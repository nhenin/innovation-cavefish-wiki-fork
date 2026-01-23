Weeks 5 and 6 continued the consolidation phase initiated with the first Work Plan (V0.1).  

The team refined the project’s positioning as an **intent-based system**, defining its technical boundaries and aligning it with the appropriate **CIP** and **SRL** frameworks.  

With the organizational foundation now in place, focus turned toward the **first implementation objective** — the *single-Service-Provider (SP) flow*.

During this period, the **cavefish-server** project was created to explore early architectural models, and the research prototype (**WBPS**) was refactored to isolate Cardano-specific logic and improve execution and logging, preparing the ground for the first functional demonstrations.


## 🎯 Goals

#### 1. Establish the foundations of the Cavefish initiative and produce a first **V0.1 Work Plan** defining objectives, structure, and validation approach.

- **Intent-Based System Definition**
  - Reframed Cavefish as an *intent-based transaction framework* rather than a pure light client.  

#### 2. [SRL 1 → 2 | CPS(s), Specification & Scoped Requirements](https://github.com/input-output-hk/innovation-cavefish/milestone/3)

- **Mapped CIPs** (Observer Scripts, Nested Transactions, Validation Zones) to define scope.  
- **Clarified value of weakly blind signatures** for trustless LC–SP collaboration.

#### 3. [SRL 2 → 3 | Single Service Provider Prototype](https://github.com/input-output-hk/innovation-cavefish/milestone/1)

- **Set up the cavefish-server project** within the repository, focusing mainly on environment initialization and the early exploration of a first architectural model.  
- **Refactored and streamlined the WBPS research prototype**, scoping it down to retain only the Cardano-specific components.

## 📄 Deliverables

- **[Cavefish Work Plan (V.0.1).pdf](https://github.com/user-attachments/files/23417315/Cavefish.Work.Plan.V.0.1.pdf)**  
- [Cavefish / Monthly Meeting – Sept 2025 Slides](https://docs.google.com/presentation/d/1-PMJGcMKq_mivWi6yxHzfwIkHed9dB7GFEmZ8JIkbEA/edit?slide=id.g2d62e322bfa_1_0#slide=id.g2d62e322bfa_1_0) 
- [Cavefish / Monthly Meeting– Sept 2025 – 4.webm](https://drive.google.com/file/d/1yIF5A5ldEjm42SDajjCkii0ex4uI86GV/view)  
- Derived **GitHub Milestones**:
  - [**SRL 1 → 2 | CPS(s), Specification & Scoped Requirements**](https://github.com/input-output-hk/innovation-cavefish/milestone/3)  
  - [**SRL 2 → 3 | Single Service Provider Prototype**](https://github.com/input-output-hk/innovation-cavefish/milestone/1)  
  - [SRL-3 → 4.5 | Cardano Testnet Multi-SP Prototype & Formal Properties](https://github.com/input-output-hk/innovation-cavefish/milestone/4)  
  - [SRL-4.5 →5 | Community Hand off](https://github.com/input-output-hk/innovation-cavefish/milestone/5)

### Commits (September 20–October 3, 2025)

* [a5ff60e](https://github.com/input-output-hk/innovation-cavefish/commit/a5ff60e76ac8b91ff52d6c221c9e45dc095d02d4) — add consumed delta to PrepareResp, since it's needed for the client to run satisfies *(Will Gould)*
* [7c45f9a](https://github.com/input-output-hk/innovation-cavefish/commit/7c45f9aae4a5d7d65cb905af3f1feafe33003205) — moved a bunch of probably-non-test stuff out of tests, added some convenience endpoints, attempted to further enhanced realism (contextually, at least) *(Will Gould)*
* [daa189a](https://github.com/input-output-hk/innovation-cavefish/commit/daa189a60c961fd2edc7c53879217179e4828fa0) — Jump to definition friendly env *(Nicolas Henin)*
* [ada334d](https://github.com/input-output-hk/innovation-cavefish/commit/ada334d5c8bb57572b9842926069091583a5b07c) — updated gitignore *(Nicolas Henin)*
* [5bb9919](https://github.com/input-output-hk/innovation-cavefish/commit/5bb99198038ba3919f37793fbfaa9d06e18273f9) — significantly fleshed out prototype, mostly "realistic" (sans proper proving, signing etc) flow working in roundtrip test *(Will Gould)*

## 🚀 Next Steps

- Prepare **Porto Workshop**.
