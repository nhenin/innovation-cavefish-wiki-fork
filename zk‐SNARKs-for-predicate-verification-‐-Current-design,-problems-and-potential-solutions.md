# :one: Why zk-SNARKs?

The core enabler of Cavefish is a WBPS protocol that enables the LC to blindly sign a transaction built by the SP while being ensured that the transaction satisfies the requested intent. For that, WBPS relies on a zk-NArg that allows the LC to verify that the predicate is satisfied (whatever the predicate is) with overwhelming probability. The ZK part allows the SP to protect the transaction from being stolen by the LC and submitted without including the corresponding fee.

To instantiate the zk-NArg, Cavefish relies on zk-SNARKs, mainly due to the wide interest the research community has had in them the last decade and the amount of libraries that are being developed. Another motivation for this is the wide and diverse toolsets that have been developed to allow capturing NP relations by easy to use programming interfaces.

Being Non-Interactive is very useful in Cavefish to let the SP producing a one-shot proof that allows the LC to take the decision of signing, without needing multiple rounds of communication and additional latency. With respect to succinctness, despite small proofs benefit the solution in terms of communication, it is not the key bottleneck. Instead, this comes from the aggregated time in generating and verifying the proof. While zk-SNARKs have typically targeted short proofs with fast verification, the prover time contributes negatively in this case because it implies a larger delay between the intent request sent by the LC and the moment when the transaction is finally signed and submitted to the blockchain.

# :two: The map of zk-SNARKs

All zk-SNARKs require some sort of public parameters to be generated first. However, there are different type of setups that can be classified according to two dimensions.

- **Transparent vs. Trusted:** determines if the public parameters of the setup must be derived from some private parameters (trusted) or if they can be directly sampled from some distribution (transparent), without private data involved. The private parameters are commonly referred to as toxic waste. If the private parameters are exposed, security is broken (it normally allows a malicious prover to forge fake proofs). The part of the setup that involves the private parameters can be executed by a single Trusted Third Party or, commonly in practice, by a set of distrustful parties running a Secure Multi-Party Computation (MPC) process, where none of the parties learns the full private value.
- **Universal vs. Circuit-Specific:** determines if the public parameters from a setup can be reused to generate proofs on different circuits or, on the contrary, a special circuit-specific setup phase must be run independently for each circuit. Note that, even when a universal setup can be reused for different circuits, some SNARKs still require to derive a proving/verifying key pair from each specific circuit to be used.

From a usability point of view, protocols with transparent and universal setups are the most versatile ones because they enable an easier handling of different circuits without re-runing setup steps or relying on private parameters. However, this normally comes at the cost of having slower proof systems. Some illustrative examples are presented below, with asymptotic costs relative to the circuit size.

| **Protocol** | **Setup** | **Prover time** | **Verifier time** | **Proof size** |
| ------------ | --------- | --------------- | ----------------- | -------------- |
| Groth16 | Trusted / Circuit-Specific | linear | linear (w.r.t. public statement x) | constant |
| Halo2 (KZG) | Trusted / Universal | quasi-linear | linear (w.r.t. public statement x) | constant |
| Halo2 (IPA) | Transparent / Universal | linear | logarithmic | logarithmic |
| Bulletproofs | Transparent / Universal | quasi-linear | linear | logarithmic |

A high-level representation of how different types of setups are used is presented in the following diagram.

![graphic view of SNARK setups](https://github.com/user-attachments/assets/6f499dd3-2e15-4b24-9757-dab10b41f585)

# :three: Predicates in Cavefish

This section introduces the specific verifications that should be done inside the zk-SNARK (namely the predicate) in the different strategies that can be adopted in Cavefish. We remark that the core verification in all strategies is abstracted `chkSpec(int_post,TxAbs)`, but its implementation varies from one strategy to another.

## :large_blue_diamond: Light Intent Strategy

In this strategy, the LC is given by the SP a `TxAbs` which is exactly the same as the actual `tx_body` to be signed but with the inputs zeroed-out (this way the LC can’t submit the `tx` itself). Thanks to this, `chkSpec(int_post,TxAbs)` can be entirely verified by the LC outside the zk-SNARK, which implies considerable resource savings. However, there are additional verifications related to the WBPS protocol that must be verified in the zk-SNARK circuit. Since this is the current strategy supported by Cavefish prototype, the pseudo-code for the different parts of the circuit is provided below.

### :small_orange_diamond: Current circuit pseudo-code

- *Provided inputs*

```
private input tx_body_cbor (byte array)
private input commitment_nonce_rho (scalar mod BabyJubJub subgroup prime)

public input tx_abs_cbor (byte array)
public input signature_nonce_comm_R (EdDSA/Ed25519 curve point provided as byte array)
public input signature_PK_X (EdDSA/Ed25519 curve point)
public input commitment_PK_EK (BabyJubJub curve point)
public input commitment_C (array of 2 field elements in BN254 Fr)
public input challenge_c (byte array of 64 bytes)
```

- *Verify correctness of transaction body*

Check that `tx_abs_cbor` matches the corresponding bytes in `tx_body_cbor`. Here, `input_ends` indicates where the public part of the transaction starts, but this can vary in a real implementation.

```
var input_ends
for i in range(input_ends, tx_len):
	tx_body_cbor[i] === tx_abs_cbor[i]
```

- *Compute transaction id*

In Cardano, what the wallet signs is the `txId`, which is the Blake2b-256 hash of `tx_body_cbor`. This is the building block where variable sized inputs (the transaction body) must be handled.
```
txId <== blake2b-256(tx_body_cbor)
```

- *Message commitment*

This phase generates the commitment of the message (i.e., `txId`) by using EC-ElGamal over BabyJubJub in ECIES mode. First, the shared secret is computed from the `commitment_PK_EK` and the `commitment_nonce_rho`. Notice that the nonce (a scalar) must lie within the prime order subgroup of BabyJubJub, which has 251 bit-length. The modulus for this subgroup is `l=2736030358979909402780800718157159386076813972158567259200215660948447373041`.

```
shared_secret <== EscalarMulAny(251)
	- with base commitment_PK_EK
	- with binary encoded scalar commitment_nonce_rho
```

Then, the key stream is generated from the shared secret by using a KDF, in this case based on Poseidon. Notice that, since the message to be commited is `txId`, which has 256 bit-length, it just suffices to encode it as two elements in BN254 scalar field $F_r$.

```
// Absorb 4 items (the EC point and the IV), Squeeze len items (len=2 for txId)
key_stream[len] <== PoseidonEx(4,len)
        - with input shared_secret
        - with input IV
```

The last step is just aplying the OTP to `txId`.

```
comm[1] <== txId_limb[1] + key_stream[1]
comm[2] <== txId_limb[2] + key_stream[2]

```

- *Compute the signature challenge*

A Cardano transaction signature uses EdDSA algorithm, i.e., deterministic Schnorr over Ed25519. However, Cavefish currently uses the randomized version due to blind signatures. Then, it must be ensured that the scalar `signature_nonce_r` lies within the prime order subgroup of the curve Ed25519. From that, the commitment of the nonce is computed by using the base point of the curve subgroup, i.e. `base_point = (15112221349535400772501151409588531511454012693041857206046113283949847762202, 46316835694926478169428394003475163141307993866256225615783033603165251855960)`. However, the circuit is directly provided with the commitment `signature_nonce_comm_R`, i.e., $R=r*G$.

The signature challenge is computed from a message that has constant size by using Sha512 algorithm (signature nonce commitment, signer’s PK and the actual message to be signed, which is `txId`).

```
msg <== R || PKSigner || txId 	// This is cte size
challenge <== SHA-512(msg)

```

- *Verification of the input values*

It must be noticed that the value held by the UTxO inputs provided in the transaction are not verified, despite they can only come from the LC otherwise the signature will be invalid in the network (we are not considering here the case of multi-signers). In fact, having an incorrect balance in the transaction will arise the same problem, i.e., on-chain transaction invalidation. What could be verified in-circuit and in ZK is nothing more than the correct format of the inputs, i.e., that they follow `transaction_input = [ transaction_id : $hash32, index : uint]`.

## :large_blue_diamond: Extended Intent Strategy

The extended intent  case is quite different because it requires `chkSpec(int_post,TxAbs)=1` to be verified completely in-circuit (because `TxAbs` does not expose anything from the private `tx_body`. This greatly increases the complexity of the circuit and can be unmanageable. In addition, the circuit must also support some cbor-parsing mechanism because the private witness is a cbor-encoded transaction (this is required by Cardano) and its fields must be decoded to achieve a proper verification.

## :large_blue_diamond: Observe Script Strategy

This strategy delegates the verification of chkSpec to an observe script that must be included in the transaction, based on [CIP-0112](https://cips.cardano.org/cip/CIP-0112#cddl) where the CDDL for the transaction body specifies that the field required_signers would change to required_observers in order to capture both KeyHash and ScriptHash credentials. However, w.r.t WBPS, the LC still needs to sign the transaction because at least one input must be included. Therefore, the circuit should verify the following:

- `txId = Blake2b-256(tx_body)`
- `C = PKE.Enc(PK, txId; rho)`
- `c = SHA-512(R, X, txId)`
- That `script_hash` is included in the `required_observers` field inside `tx_body` (in principle, it could be done by bytestring comparison, but special care must be taken into account if this is exploitable w.r.t. how transactions are cbor-encoded).

# :four: Main challenges and potential solutions

## :question: (C1) The cost of binary hashing

Hashing is the main bottleneck in Cavefish, more specifically with respect to the circuit size (number of constraints) and therefore prover time. The main reason for this is that bit/byte-oriented hash algorithms are very inefficient in algebraic-oriented protocols like zk-SNARKs because constraining binary operations (actually a large number) imply a large underuse of the real size of the field (typically in the order of 256 bits per field element).

Cavefish needs to compute two binary hashes: `Blake2b` for `txId` and `SHA-512` for the Schnorr challenge. Both of them are quite inefficient, but while the latter has a length-fixed and (somewhat) small input (`R||X||txId`), the former has the problem of being applied to the whole `tx_body`, which can vary between 84 and 15036 bytes (see also section C2 for the problem of inputs with variable length).

The current prototype implementation in Cavefish is based on a Groth16 zk-SNARK by using Circom and SnarkJS. In this case, there is no substantial optimization for the hashing algorithms and this is translated into poor performance in proof generation, highly impractical when reaching the largest transaction lengths (ref to benchmarks).

Some rough tests have also been done with Midnight-zk, a Halo2-based zk-SNARK library. The benefit for this is that Plonk supports lookup arguments that can significantly improve specific applications. In this case, both `SHA2`and `Blake2` gadgets rely on lookup tables to speed up the proof generation. However, despite this is translated into a better scaling for large-length transactions, there is a base constant cost to compute the lookup tables that harms the cases where transactions are small. It is fair to mention that this approach scales well with powerful hardware.

- The paper [[zk-bench-paper](https://eprint.iacr.org/2023/1503.pdf)] provides evidence that for large inputs (`2^14` bytes), some zk-SNARK libraries are not very efficient for SHA2 (despite using an AWS m6i.8xlarge instance of 32 vCPU of 3.5 GHz). In fact, SnarkJS is the most inefficient one.
- [[zk-Harness](https://www.zk-bench.org/circuit)] also provides benchmarks for SHA2 circuit with Groth16.

## :heavy_check_mark: (C1) Potential solutions

### :small_orange_diamond: Optimizations to hash implementation

One approach would be to optimize more the hash function gadgets implementations. However, this is very dependent on the hash algorithm itself and the proof system/frontend interface, e.g., if using R1CS or Plonkish and the availability of additional techniques such as lookup tables. While some performance can be gained by this path, it is not expected to be very significant with respect to the actual times.

### :small_orange_diamond: zkVMs

zkVMs are solutions that target general-purpose computations, by designing proof systems that take as input VM execution traces (which are internally compiled into arithmetic constraints). This enables a model where the efficiency of some programs in the target architecture of the VM is (relatively) translated to the efficiency of the proof system w.r.t. the heavy overload introduced by SNARKs that are better suited for large field computations. Because of this, some zkVMs can obtain better performance in binary-oriented computations. However, it must also be noted that while proving a sha2 or blake2 hash could be speeded up, other computations involved in Cavefish related to elliptic curves could be slowed down.

- **Some benchmarks**

The resource [[zk-bench](https://zkbench.dev/)] provides some results for different hash functions in different zk-VMs. It can be seen that Risc Zero is the most optimised for SHA2, providing good performance (< 10s) even with large inputs (10kB) in a 16x CPU setting. The other zk-VMs work in different orders of magnitude.

In [[zk-benchmark](https://github.com/delendum-xyz/zk-benchmarking?tab=readme-ov-file#prover-performance)] there are also SHA2 prover times for both Risc Zero and MidenVM. However, the inputs are only given for sizes of 10 and 100 bytes. In this case, Risc Zero can beat the < 10s target with an Apple M2 CPU. MidenVM also provides times for Blake3, which should be expected faster than Blake2.

- For Risc Zero, it must take into account if the provided proof is a composite one. If so, compressing into a succinct or groth16 proof would include additional prover delays.

### :small_orange_diamond: Folding

Folding is an advanced technique that allows to linearly combie (“fold”) multiple instances of the same relation into a single one, such that only the final one is verified in the zk-SNARK while still guaranteeing the knowledge soundness.

Folding can be seen as a protocol with two phases: (1) a cheaper pre-processing where all the desired instances are folded, i.e., accumulated into a single one, and (2) a more expensive final step where the proof is generated for the accumulated relation. This can provide savings in the prover time when it needs to repeat the same computation multiple times because of the cheaper folding step.

In the case of Cavefish, folding would help when using a Merkle-Damgard-based hash algorithm such as sha2 or blake2, where there is a (typically costly) compression function which is applied iteratively on fixed-length chunks of the input message, i.e., `st_i = CompressF(st_(i-1), m_i)` for round `i`. In this case, the relation would capture CompressF and would be applied sequentially by folding `R_acc(i-1) X R_i → R_acc(i)`. It can be noticed that this is in fact an Incrementally Verifiable Computation (IVC) but, instead of producing sequential proofs where each one verifies the previous one (and where only the last one would be provided to the verifier), i.e., where there would be no real savings in the prover time, folding allows to achieve this IVC more efficiently in time.

It must be noted that folding is quite a novel technique and the availability of libraries is still not wide. However, there are a few implementations such as [Nova](https://github.com/microsoft/Nova) or [Sonobe](https://github.com/privacy-ethereum/sonobe), that would help in determining experimentally the impact of folding in this use case.

The following benchmarks have been obtained in WSL, i7-13620H 10-cores (2.40 GHz), 16 GB RAM.

- **Benchmark: SHA256 in Nova** (Only folding steps)

| **Num. of blocks in message** (1 block = 64B) | **Prover time** (s) |
| --- | --- |
| 1 | 0.065 |
| 2 | 0.071 |
| 4 | 0.098 |
| 8 | 0.131 |
| 16 | 0.222 |
| 32 | 0.381 |
| 64 | 0.642 |
| 128 | 1.201 |

- **Benchmark: Compressed SNARK in Nova** (Executed once at the end)

| **Num. of constraints in step circuit** | **Prover time** (s) |
| --- | --- |
| 0 | 2.15 |
| 6399 | 3.04 |
| 22783 | 3.45 |
| 55551 | 4.39 |
| 121087 | 6.32 |
| 252159 | 10.33 |
| 514303 | 16.83 |
| 1038591 | 35.10 |

### :small_orange_diamond: Split hashing

Given a Merkle-Damgard-based hash algorithm, split hashing is a technique where, given a message that requires more than one input chunks, part of the chunks are processed inside the zk-SNARK circuit while the rest are processed outside of it, therefore saving proving time for those computed outside.

To be more specific, assume an `n-chunk` message `[m_1, …, m_n]` where the first 2 chunks contain private data that cannot be disclosed to the verifier (for example, the transaction inputs that must be hidden to the LC in Cavefish). Given that the LC knows the full content of those chunks in `[m_3, …, m_n]`, the strategy in split hashing is letting the LC verify the hash for those chunks outside by making a sort of length extension attack. For this, the LC is given the hash state update `st_2 = CompressF(st_1, m_2)` and from this, it can compute the rest of the MD updates until the final hash is obtained. Then, the first two updates (to obtain `st_1` and `st_2`) are verified inside the circuit with the two first chunks of the message provided as witness.

This technique is applied easier when the private chunks are consecutive provided, and mostly when they are at the beginning or the end of the message, because a single intermediate state st_i is provided as public input to the circuit (by the LC). When multiple sparse chunks contain private information, all the states between `private-public` or `public-private` must be provided as inputs to verify correctness of the final result.

**Security considerations**

- This strategy relies on the assumption that the compression function is hard to invert, which is the exact assumption applied to an MD construction. It should rest to prove formally that the adversary can’t obtain the input chunk `m_i` from an intermediate state result `st_i`. For example, in the case of blake2, it is claimed in [[LMN16](https://eprint.iacr.org/2016/827.pdf)] that its compression function is indifferentiable from a random function in a weakly ideal cipher model.
- Another caveat related to an implementation aspect is the format of the private input. The base assumption would be to let the whole chunk `m_i` to be fully private to the adversary. However, in the case of Cavefish (when using a light intent strategy), the private inputs don’t necessarily fit a single (or multiple) chunks, but instead they can lie in the middle. Even if this doesn’t affect the security of the compression function itself, it would allow the adversary to reduce the sampling message space to brute-force one intermediate state, depending on the amount of information known in each chunk. As an illustrative example, assuming blake2b 128-bytes input chunks, if the inputs of total length 128-bytes lie in the middle of two chunks, the adversary would need to brute-force two independent 64-byte instances instead of a single 128-bytes one.

### :small_orange_diamond: Unequal hashing

This solution is directly based on [[KVW25](https://eprint.iacr.org/2025/1500.pdf)]. The general concept is to use different hash functions on the same message and be able to verify that the outputs (different ones) come from the same message. A brief description of the protocol would be the following:

- Given two different hash functions `H_1` and `H_2`, and given the respective hash images computed from a message `m`, i.e., `h_1 = H_1(m)` and `h_2 = H_2(m)`, the authors provide an interactive protocol that allows to verify that `h_1` and `h_2` come from the same message `m`.
- The protocol works as follows:
   - Alice computes `h_1` and sends it to Bob.
   - Bob computes `h_2` and sends it to Alice.
   - Alice computes `gamma_1 = UHF(h_1 + h_2, m)` and sends it to Bob.
   - Bob computes `gamma_2 = UHF(h_1 + h_2, m)` and sends it to Alice.
   - They both accept if `gamma_1 = gamma_2`.
- To use unkeyed hash functions `H_1` and `H_2` from different families the security is based on the Random Oracle Model.
- The UHF in the paper is modeled as a polynomial of degree `k`, where `k` is the size of the inputs in field elements.
- There is a lower bound for the field size in order to preserve the target security parameter `lambda`, which is: `|F| >= k * 2^(2*lambda)`. Otherwise, UHF-Hardness is not achieved.

The application to Cavefish would be very straightforward and would allow to use blake2b as normally and move to Poseidon (a SNARK-friendly one) inside the circuit, while still having soundness guaranteed. To be more precise:

- Compute `txId = blake2b-256(tx_body)` externally and provide it as input to the zk-SNARK circuit (with `txId mod |Fr|`, maybe loosing a couple of bits).
- Compute `txId’= Poseidon(tx_body)` inside the zk-SNARK circuit.
- The UHF is expected to be computed both inside and outside the zk-SNARK, while verification of `gamma_1 = gamma_2` is done inside the zk-SNARK (note that the external gamma implies that the Poseidon hash function must also be computed outside of the circuit, but this is fast in general-purpose CPU).
- The max size of a Cardano tx is 16 KiB, therefore around `k=518` field elements in `F^254` (e.g. for BN254).
   - This implies around `lambda=122` bits of security with BN254.
   - More bits of security could be achieved with a larger field, e.g., in BLS-318.

## :question: (C2) Handling variable length inputs

zk-SNARKs rely on arithmetic circuits that must be fully evaluated in a deterministic way. This implies a set of considerations that really affect usability and performance, overall in solutions such as those from the privacy ecosystem which already are inefficient for some use cases. For example:
- **Circuit structure** is modified together with the length of the input, directly in the number of wires for the public input and witness, and indirectly in the amount of computation (e.g., number of rounds or size of internal structures) that is needed by the functions used inside the circuit. This constraints the use of hardcoded parameters.
- **Conditional branching** is not available in an efficient way. Since the circuit evaluation is deterministic, all branches have to be evaluated and the desired result must be filtered out based on some arithmetic trick using selectors. This is more complex to handle when the selected branch must be kept private, because the implementation can leak information about this if not implemented correctly.

### :small_orange_diamond: The impact of setup phases

In the case of zk-SNARKs, modifying the circuit structure may imply quite a large restriction in usability because of the need of setups, mostly when they are trusted. Proof systems such as Groth16 achieve efficient and compact proofs, but requires two setup phases with toxic waste which are slow if centralized and overall if run by some MPC. Therefore, modifying the circuit on demand is not realistic in this case. In other systems such as Plonk, using a universal setup makes this easier to handle, but there is still a need of computing the prover/verifier keys for each specific circuit. This last step can still be inconvenient in systems where many different circuits are needed or there is not time enough for computing the keys each time (or having not enough resources available).

## :heavy_check_mark: (C2) Potential solutions

### :small_orange_diamond: VarLen strategies in circuits

Handling variable length inputs inside arithmetic circuits is complex and typically introduce quite large overheads. From a general perspective, there are three main solutions:

1. Forcing the input length to be used to a specific invariant length and using padding strategies if required, and where the circuit always computes the same amount of steps. This option is not very realistic, mainly when the zk-SNARK is replicating an external computation that cannot be updated to a scenario with padding.
2. Fixing fine grained cases for different lengths and having a specific circuit to be used for each one of the cases. This can be useful when the set of possibilities is not too big such that the system can rely on having N circuits with their respective independent setups and prover/verifier key pairs. However, if the variability of the input lengths is large, N can raise in an unmanageable way.
3. Using a buffered approach. In this case, a single circuit is designed to take as private input a single buffer up to a length Lmax. All inputs that are smaller can be provided with the corresponding padding. However, instead of fixing the computation to process the whole input buffer, the circuit is also provided a private input that captures the real length of the input. This allows the circuit to only process the useful amount of data inside the buffer and obtain the desired results. However, due to different caveats, this approach typically implies some overhead in processing the dummy data too (even if the circuit manages to discard the unwanted results).

From the three cases, (3) is the more realistic one for most of the applications. However, the circuit complexity is worst case, i.e., it depends on Lmax. Therefore, it should not be discarded the option to combine (2) and (3) in a way that a nice trade-off is found between the number of circuits to be used and the complexity in each of them. One could select, for example, to use 3 circuits such that:

- Circuit 1 handles inputs of length `(0,L)`
- Circuit 2 handles inputs of length `(L,2L)`
- Circuit 3 handles inputs of length `(2L,Lmax)`

### :small_orange_diamond: The case of hashing

Hash functions typically rely on some sort of chunk solution, where data is padded until a multiple of the chunk length CL is achieved. Then, the number of chunks N determine the number of rounds to be executed. To be more precise, blake2 and sha2 are [Merkle-Damgard constructions](https://en.wikipedia.org/wiki/Merkle%E2%80%93Damg%C3%A5rd_construction).

To work with variable sized data, the circuit must handle an input buffer B with a fixed number of chunks Nmax, where the desired data of length `l` is inserted in `B[0:l]` and the rest of the buffer is filled with dummy data. This implies that the circuit can only process data of length less or equal than `CL * Nmax` (this limit is a key design constraint) and that all chunks in `B` will be processed by the hash block, otherwise a modification in the circuit structure would be implied.

To process the desired hash result, the circuit must be provided with another input that captures the real data length `l` (can be private or public depending on the use case). Then, based on `l`, the circuit will discriminate which chunks are actually contributing to the final result and which have to be discarded. For this, there are two main strategies, depending on the use of the hash function gadget as a black box (less efficient) or a specific design for this purpose (more efficient).

### :small_red_triangle: Example: hashing as a black box

For this strategy, it is important to notice that, if the hash function needs to process `i` chunks, it requires to sequentially update the state at least `i` times (let’s call this `Hi`).

- *Hashing phase*

This strategy implies to execute the hash function `N` times, each time considering one additional chunk of the input, i.e., `(H1,H2,…,HN)`. Notice that chunki will be processed `i` times, so there will be a large waste of resources. Then, each hash block will produce a result `ri=Hi(chunk1||…||chunki)`.

- *Conditional selector phase*

The final step is selecting the desired result `ri` from the set of all `N` results obtained. To do so, the standard arithmetic trick is to use a binary selector `s` of length `N` and where only 1 digit is set to 1 (this must also be checked inside the circuit). Then, the desired result is computed as `sum_i (si * ri)`, where all but one will be filtered out.

- *Cost*

Let’s assume that the hash function cost is dominated by a compress function that works on each chunk at a time. Since the step `Hi` implies compressing the first `i` chunks, we have the following cost: `compress(chunk1)*N + compress(chunk2)*(N−1) + ... = (N*(N+1))/2*compress = O(N2)`.

### :small_red_triangle: Example: hashing with conditional state updating

This strategy implies to work at the internal hash function level. The idea is to sequentially compress all chunks and only contribute to the final state if enabled by the binary selector. Then, for each chunk `i`:

- Compute state `sti=compress(sti−1,chunki)`
- Output the new state if the chunk is included, otherwise output the previous state, i.e, `st_out=st_(i−1)+s_i*st_i` (note that `si` will be 0 for all dummy chunks).

- *Cost*
Now, there is a sequential execution of the hash function internals (the compression) and therefore the cost is `O(N)`.

### :small_red_triangle: Example: SHA2 vs. Blake2

It is clear that hashing as a black box can be applied to any hash function, but it is not the same for hashing with conditional state updating, or at least it requires modifications.

In the case of Cavefish, SHA2 is a pure MD-based hash, while Blake2 is a HAIFA, i.e., an MD enhanced with additional features: round counter, (optional) salt and finalization tag. This difference also conditions how message padding is treated and therefore must be handled by the circuit.

- *Handling the padding*

Than natural way to achieve a VarLen hash with conditional state updating is to provide the input already padded in the last chunk and with additional dummy chunks until reaching `Nmax`. This way, the circuit will be able to apply the strategy as described above directly.

```
input = [data_chunk_1 .. data_chunk_(n-1)] || [data_chunk_n_padded] || [dummy_chunks]
```

However, conditioned by the difference between MD and HAIFA, SHA2 and Blake2 add the padding differently:

- **SHA2** adds the padding as a suffix to the message: `msg_bits||1||0..0||<msg_len>`, where the length tag is a 64-bit big-endian value containing the actual length of the message. The number of 0s in the middle are added s.t. the final padded message is a multiple of 64 bytes.
- **Blake2** adds zero padding until the message reaches a length multiple of 128 bytes. However, at each compression step, a counter that keeps track of the number of bytes already hashed is added to the state. Therefore, if the last chunk is not complete, even if its internally padded to zeroes, the last update of the counter will add less than 128 to the added value.

To illustrate the problem:

```
# Blake2b with internal padding:
last_block = [1, 2, 3]
ctr = ctr + 3
padded_last_block = [1, 2, 3, 0, 0, ..., 0]
h = compress(h, last_block, ctr, final)

# Blake2b with external padding:
last_block = [1, 2, 3, 0, 0, ..., 0]
ctr = ctr + 128
padded_last_block = last_block
h = compress(h, last_block, ctr, final)
```

From the previous, it can be seen that providing the input already padded for SHA2 suffices, while in Blake2, if the input is externally padded it will affect the last value of the counter added to the state. 

Because of this, some additional mechanism must be included such that it enforces the last update of the counter to be the correct one (according to the real number of bytes hashed) even if the message has been externally padded. This can be done by computing the remaining bytes of the last block in advance and providing that value to the compression function when it corresponds, but this adds complexity to the circuit because the counter computation must be constrained as a witness.

- *Handling the finalization tag*

Another problem of the HAIFA is that the last chunk processed has added a finalization tag which is different from the rest of the chunks. This is done to avoid length extension attacks. However, for our conditional state updating strategy, it implies that apart from adding blocks only if they contribute, there is an additional conditioning (another branch) for the last one. Therefore, in a deterministic scenario, this implies treating all chunks in two potential paths: as if there are more chunks to add or if they are the last one. Generally speaking, each round would compute:

```
st_out = st_(i-1) + s_i * [(1-s'_i)*[st_(i),last_0] + (s'_i)*[st_(i),last_1]]
```
For the sake of completeness, a rough and high-level pseudo-code is provided for VarLen Blake2b:

```
// Message = [message_full_chunks] || [message_last_chunk_padded] || [dummy_chunks]
// Must provide as private input a selector s[N_MAX] with only 1 bit set (others must be 0)
// Must provide as private input the actual message length m_len

// Initialize state
hash_state[N_MAX]      // The circuit will compute N_MAX states as if each of them were the last one
hash_state[0] <-- IV

// We omit the mix key phase

// Strategy: compute compress for each block conditioned by being last block or not
// The last block flag is actually the input selector s[N_MAX]
// The offset counter (oc) must be adapted to the last block
var last_offset = m_len mod 128

for (var i = 0; i < N_MAX; i++) {
	oc = (1 - s[i]) * ((i+1)*128) + s[i] * last_offset
	// CompressionF is given the offset counter and last block flag
	compr[i] = CompressionF()
	// Adapt data structure (any low-level adaptation needed)
	blocks[i] = ...
	// Apply the compression function
	compr[i].h <-- hash_state[i]
	compr[i].m <-- blocks[i]
	compr[i].oc <-- oc
	compr[i].s <-- s[i]
	compr[i].out --> hash_state[i+1]
}

// Convert to hash words or any other format needed

// Filter the desired result
res = \sum_i hash_state[i] * s[i]
```