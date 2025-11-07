## 🧭 Summary

Weeks 6 and 7 were slower in terms of development activity, as the team participated in the **Innovation Workshop in Porto**.  
This period focused on presenting the Cavefish initiative to the broader innovation group and syncing closely with the cryptography team.  
Discussions centered on challenging the current stack and aligning on the next phase of development.  
It was decided that the Cavefish team will adopt **Halo2** as the new proof system, replacing **Groth16**, and will likely transition to a different **setup ceremony** than the one used by the research team.

This marks a gradual shift from rapid prototyping toward a **production-oriented implementation**:  
while Circom and Groth16 remain ideal for early iterations, future versions will use Halo2 and an alternative setup to enhance performance and scalability.

Following organizational updates, the team also agreed to target a **100-day demo milestone** by the end of November.


## 🎯 Active Milestones

#### 1. [SRL 1 → 2 | CPS(s), Specification & Scoped Requirements](https://github.com/input-output-hk/innovation-cavefish/milestone/3) 

- **Intent Representation & Observer Integration**
  - Confirmed Ledger team’s Observer timeline and compatibility with Cavefish DSL plans.

#### 2. [SRL 2 → 3 | Single Service Provider Prototype](https://github.com/input-output-hk/innovation-cavefish/milestone/1) 

- **Cryptographic Stack Analysis**
  - Compared Groth16 vs Alternatives for setup trust, efficiency, and universality.  
  - Challenging Circom vs Halo2 (recommended by the cryptographer team).  
- **Circuit Complexity & Parsing Challenges**
  - Identified transaction parsing in SNARKs as a major bottleneck.  
  - Tested simplified “blob-of-bytes” approach with risk assessment for malicious SP cases.

#### 3. [SRL 1 | Create Workstream Briefing Pack](https://github.com/input-output-hk/innovation-cavefish/milestone/3) 

- First draft – Workstream Briefing Pack.  
- First draft – Explainers covering basic concepts and goals of workstream.  
- First draft – Short non-technical article for Essential Cardano / Cardano Forum.

### Preparing Workshop
- Slides + Innovation Group presentation.

## 📄 Deliverables

### 🧩 Documents 

- [Cavefish Introduction @ Porto Workshop](https://docs.google.com/presentation/d/1qd04seyx5nXui0811PuORWFwzfAYf-UaXyNLnkwPVPY/edit?slide=id.g38c64c49422_0_637#slide=id.g38c64c49422_0_637)  

### 🧩 Commits (October 4 – 15, 2025)

Below is the list of commits merged into `main` during this period:

* [4d92fa3](https://github.com/input-output-hk/innovation-cavefish/commit/4d92fa3953b9f785e3836a0d650cbe78d304a5d2) — **executable** *(Will Gould)*
* [9eff136](https://github.com/input-output-hk/innovation-cavefish/commit/9eff1360c8bbafb4acad9729bcb92aaa7d6a97cf) — **TxAbs encoding**, roundtrip test exercising REST endpoints, logbook *(Will Gould)*
* [35ffbc9](https://github.com/input-output-hk/innovation-cavefish/commit/35ffbc9fdbbe979cab7e72c7b5cd97ee4de99e81) — updated documentation *(Nicolas Henin)*
* [43852f7](https://github.com/input-output-hk/innovation-cavefish/commit/43852f7e0a89f4f5b72d5d5960888da7628dd205) — restructured files and folders *(Nicolas Henin)*
* [a755191](https://github.com/input-output-hk/innovation-cavefish/commit/a755191ba15dcf5f7b1a9d4c90e531f9b5e16233) — updated `.gitignore` *(Nicolas Henin)*
* [666da78](https://github.com/input-output-hk/innovation-cavefish/commit/666da78b7bcd0cf0fcdee0dfb2951d5f4f5055f1) — added input generator *(Nicolas Henin)*
* [0b37477](https://github.com/input-output-hk/innovation-cavefish/commit/0b374771b380b616ed078a55e65f4b1768c51c79) — update `.gitignore` for **wbps_circuit artifacts** and `node_modules` *(Nicolas Henin)*
* [e8242ac](https://github.com/input-output-hk/innovation-cavefish/commit/e8242ac6284075e1aa3c800418a6751bd9df1535) — `make` creates a `/build` folder; all temp files are now produced in it *(Nicolas Henin)*


## 🚀 Next Steps
- Preparing **Work Plan for 100-days demo**.