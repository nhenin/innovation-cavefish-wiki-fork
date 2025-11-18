This document explains and implements the steps

```text
x ← Z_q ;  X ← g^x
r ← Z_q ;  R ← g^r
```

in the BN254 group used by our WPBS / Cavefish protocols.

- `(x, X)` is the **long-lived WBPS keypair**  
  - `x` – WBPS secret key (scalar in BN254 field `Fr`)  
  - `X` – WBPS public key (point in BN254 group `G1`, `X = g^x`)

- `(r, R)` is the **ephemeral Schnorr nonce pair**, generated **per protocol session**  
  - `r` – Schnorr nonce secret scalar (never reused across sessions)  
  - `R` – Schnorr commitment point (public, `R = g^r`)

Here:

- `g` is the generator of `G1`
- `q` is the group order (the modulus of `Fr`)
- `Z_q` is the set `{0, 1, …, q − 1}` (we sample from `{1, …, q − 1}`)

The relation `X = g^x` and `R = g^r` is elliptic-curve scalar multiplication, not regular integer exponentiation.

We show concrete implementations in:

- JavaScript (Node / browser, using `@noble/curves`)
- Haskell (using BN254 bindings)

The goal is to give **only the computation**, not impose any particular design.

## 🔵 JavaScript implementation (BN254 – @noble/curves)

This version uses the `@noble/curves` BN254 implementation and Node's `crypto` RNG.
It works in Node and can be adapted to the browser.

```js
import { bn254 } from "@noble/curves/bn254";
import { randomBytes } from "crypto";

// Utility: sample a non-zero scalar in Z_q
function randomScalar() {
  const q = bn254.CURVE.Fr.ORDER;

  let k;
  do {
    // 32 random bytes -> BigInt
    k = BigInt("0x" + randomBytes(32).toString("hex"));
  } while (k === 0n || k >= q); // reject 0 and values outside [1, q-1]

  return k;
}

// 1. Long-lived keypair: (x, X)
export function generateLongLivedKeypair() {
  // x ← Z_q
  const x = randomScalar();

  // X = g^x in G1
  const G = bn254.G1.ProjectivePoint.BASE; // generator g
  const X = G.multiply(x).toAffine();      // { x: bigint, y: bigint }

  return { x, X };
}

// 2. Ephemeral nonce pair: (r, R)
export function generateNoncePair() {
  // r ← Z_q
  const r = randomScalar();

  // R = g^r in G1
  const G = bn254.G1.ProjectivePoint.BASE;
  const R = G.multiply(r).toAffine();      // { x: bigint, y: bigint }

  return { r, R };
}
```

What this provides:

- `generateLongLivedKeypair()`  
  - returns `{ x, X }`  
  - `x` is stored long term (WBPS secret key)  
  - `X` is sent to the SP and stored as part of `vk`

- `generateNoncePair()`  
  - returns `{ r, R }`  
  - `r` is used only within a single signing / WPBS session  
  - `R` is the public Schnorr commitment sent to the SP  

You should never reuse the same `r` across two different sessions.

## 🟢 Haskell implementation (BN254 bindings)

Below is the same logic expressed in Haskell, assuming BN254 bindings that expose:

- `Fr` as the scalar field
- `G1` as the elliptic-curve group
- `gen` as the G1 generator
- `randomFr` as a uniform sampler over `Fr`
- `mul` as scalar multiplication

```haskell
module Cavefish.Keys
  ( KeyPair(..)
  , NoncePair(..)
  , generateKeyPair
  , generateNoncePair
  ) where

import           Crypto.Pairing.BN254  -- or equivalent BN254 module

-- | Long-lived WBPS keypair: (x, X)
data KeyPair = KeyPair
  { sk :: Fr   -- ^ secret scalar x
  , pk :: G1   -- ^ public point X = g^x
  }

-- | Ephemeral Schnorr nonce pair: (r, R)
data NoncePair = NoncePair
  { nonceSk :: Fr  -- ^ nonce scalar r
  , noncePk :: G1  -- ^ commitment R = g^r
  }

-- | Draw a random scalar in Fr.
randomFrIO :: IO Fr
randomFrIO = randomFr  -- placeholder: use the RNG function provided by your library

-- | Compute (x, X) with X = g^x.
generateKeyPair :: IO KeyPair
generateKeyPair = do
  x <- randomFrIO              -- x ← Z_q
  let g  = gen                 -- G1 generator g
      X  = g `mul` x           -- scalar multiplication: X = g^x
  pure KeyPair { sk = x, pk = X }

-- | Compute (r, R) with R = g^r for a single session.
generateNoncePair :: IO NoncePair
generateNoncePair = do
  r <- randomFrIO              -- r ← Z_q
  let g  = gen
      R  = g `mul` r           -- R = g^r
  pure NoncePair { nonceSk = r, noncePk = R }
```

If your BN254 library uses different names, the mapping is:

- `Fr`       → scalar field (your library may call it `Scalar` or `Fr`)
- `G1`       → group element type (e.g. `G1Projective` / `G1Affine`)
- `gen`      → group generator (sometimes `generator` or `g1Generator`)
- `mul`      → scalar multiplication
- `randomFr` → function that returns a uniform random scalar

The essential pattern is:

```haskell
x <- randomScalar    -- x ← Z_q
X <- scalarMul g x   -- X = g^x

r <- randomScalar    -- r ← Z_q
R <- scalarMul g r   -- R = g^r
```

## 🧠 How this fits in the Cavefish / WPBS flow

In your updated registration and signing flows:

- During **User.Setup**  
  - The client computes `(x, X)` using `generateLongLivedKeypair`.  
  - It sends `X` (and userPublicKey) to the SP.  
  - It keeps `x` locally and never sends it to the SP.  
  - The SP stores `X` inside its server-side `vk` structure:  
    `{ publicVerificationContext, userPublicKey, X }`.

- During each **WPBS session / signing attempt**  
  - The client generates a fresh `(r, R)` pair using `generateNoncePair`.  
  - `r` goes into the Schnorr response `s = r + c·x (mod q)`.  
  - `R` is the public Schnorr commitment referenced in the statement and signature.

So:

- `(x, X)` is the **long-lived identity keypair**.  
- `(r, R)` is an **ephemeral per-session nonce keypair** that must never be reused.

This text and code can be dropped directly into a wiki page to document
how to correctly compute both pairs in BN254, in both JavaScript and Haskell.
