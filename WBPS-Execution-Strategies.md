# ⚙️ Summary

This page explains the **two trustless execution strategies** currently under active development for running the **Weakly Blind Predicate Schnorr (WBPS)** protocol.  
Both strategies maintain the cryptographic guarantees of WBPS but differ in **abstraction level**, **execution model**, and **user interaction pattern**.  

The first strategy is based directly on the **academic specification** introduced in the Cavefish paper — it operates at the **transaction level**, focusing on minimal assumptions and maximal transparency.  

The second strategy is a **new proposal from the development team**, designed specifically for **Cardano**, and leverages the upcoming **Observer Script** feature to move the same logic to a **higher abstraction layer**.  

This design makes use of Cardano’s unique capabilities to simplify the user experience while preserving verifiability and decentralization.

## 🧩 Overview

Both strategies are **trustless**, but differ in *where* verification occurs:
- The **Tx-Level Strategy** operates directly on raw Cardano transactions (low-level).  
- The **Observer-Script-Level Strategy** uses an audited script abstraction to enforce the same guarantees (high-level).

| Aspect | **Option 1 — Tx-Level Strategy** | **Option 2 — Observer-Script-Level Strategy** |
|--------|----------------------------------|-----------------------------------------------|
| **Abstraction Level** | Low (direct transaction semantics) | High (observer script abstraction) |
| **Verification Point** | User validates transaction directly | Script enforces properties on-chain |
| **Trust Model** | Fully trustless (user-controlled) | Fully trustless (script-verified) |
| **User Focus** | Cardano transaction structure and logic | Intent definition and script selection |
| **Transaction Fees** | Lower (no script execution) | Higher (script execution costs) |
| **Transparency** | Maximum — user verifies all details | Delegated — script encapsulates logic |
| **Usability** | Requires expert understanding | Accessible to all users |
| **Maintenance** | Complex — per-use verification | Centralized — reusable verified scripts |
| **Cardano Feature Used** | Standard transactions | **Observer Script** (upcoming feature) |

## ⚙️ Option 1 — Tx-Level Strategy

### Description
This strategy follows the **academic model** proposed in the original Cavefish paper.  
It executes WBPS directly at the **transaction (Tx) layer**, where the **Service Provider (SP)** constructs and provides a verifiable transaction that the **Light Client (user)** can inspect and validate before signing.

### Workflow
1. SP produces the transaction (TxAbs) representing all constraints and commitments.  
2. The user reviews and verifies that it enforces the intended properties.  
3. The proof is verified locally before the user signs the transaction.  

### ✅ Advantages
- **Lowest transaction cost** (no script execution).  
- **Maximum transparency** — every component can be reviewed by the user.  
- Ideal for **advanced users**, auditors, or formal verification tooling.  
- Provides a **ground truth** implementation aligned with the academic model.

### ⚠️ Limitations
- Requires understanding of **Cardano transaction internals**.  
- Verification complexity grows with transaction logic.  
- Not suitable for general users or lightweight applications.  
- Tooling and visualization layers are necessary for non-experts.  

## 🔐 Option 2 — Observer-Script-Level Strategy

### Description
This approach is a **Cardano-specific extension** developed by the team.  
It introduces an additional **abstraction layer** where WBPS logic is executed and verified within a **trusted on-chain Observer Script**.  
The script enforces the same correctness conditions defined in the Tx-level approach, but in a reusable and human-readable way.

### Workflow
1. The user defines their intent via the Light Client UI.  
2. The SP uses a verified **Observer Script** that enforces WBPS constraints automatically.  
3. The user signs only the intent, not the raw transaction.  
4. The script ensures trustless validation on-chain, leveraging Cardano’s ledger semantics.

### ✅ Advantages
- **Higher abstraction** — users interact with intents, not raw transactions.  
- **Improved UX** — ideal for web and mobile frontends.  
- **Reusable, audited scripts** can gain **community reputation** and become standard modules.  
- Enables new patterns for **trustless DApp interoperability**.  

### ⚠️ Limitations
- **Higher transaction fees** due to script execution.  
- **Dependent on community auditing** and script correctness.  
- Transparency becomes **indirect** (trust shifts to code reputation).  
- Requires **Observer Script support** on-chain (upcoming feature).  

## ⚖️ Summary: Trade-Offs

| Category | **Tx-Level Strategy** | **Observer-Script-Level Strategy** |
|-----------|----------------------|------------------------------------|
| **Transparency** | Full — user verifies manually | Delegated — script verified |
| **Abstraction** | Low-level | High-level |
| **User Cognitive Load** | High | Low |
| **Transaction Fee** | Minimal | Moderate to high |
| **Implementation Scope** | General blockchain agnostic | Cardano-specific |
| **Ideal Use Case** | R&D, formal verification, advanced users | End-user applications and DApps |
| **Maturity Path** | Baseline proof-of-concept | Production-ready design target |

## 🚀 Future Outlook

Both strategies are essential components of the WBPS roadmap:  
- The **Tx-Level Strategy** will serve as the **reference implementation** for correctness and benchmarking.  
- The **Observer-Script-Level Strategy** will evolve into a **production-grade execution model**, leveraging Cardano’s scripting features for scalability and usability.  
- Over time, **audited Observer Scripts** could form an open standard — allowing users to safely rely on pre-verified logic while still maintaining full on-chain verifiability.
