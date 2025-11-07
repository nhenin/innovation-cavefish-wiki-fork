## 🧭 Summary

During these two weeks, the Cavefish initiative moved from architectural alignment to active prototype development.  
The focus was on **refactoring**, **project modularization**, and **integrating the WBPS wrapper** into a functional end-to-end flow.  
The team also began defining the structure of the upcoming **“100-ish Days Demo”**, scheduled for early December, and discussed feasibility spikes to validate **zero-knowledge verification in constrained environments**.

Polina Vinogradova submitted a **Cardano Improvement Proposal (CIP)** introducing a new **Intent-Based Transaction Processing model** — a significant step toward aligning ledger behavior with Cavefish’s design goals.  
Meanwhile, the engineering team worked on **sub-project separation**, Nix reproducibility, and CI improvements to prepare for the next milestone.

## 🎯 Goals

#### [SRL 1 → 2 | CPS(s), Specification & Scoped Requirements](https://github.com/input-output-hk/innovation-cavefish/milestone/3)

- **Intent Representation & Ledger Alignment**
  - Submitted the new **Intent-Based Transaction Processing CIP** ([CIP #1110](https://github.com/cardano-foundation/CIPs/pull/1110)) by *Polina Vinogradova*.  
    - Rendered proposal: [CIP — Intent-Based Transactions (Draft)](https://github.com/polinavino/CIPs/blob/polina/light/CIP-%3F%3F%3F%3F/README.md)
  - The proposal introduces a mechanism requiring users to sign both the **transaction ID** and the **hash of guard scripts**, enabling **intent-based validation** and off-chain verifiability.  
  - Continued discussions on the conceptual separation between **intent (observer script)** and **fulfillment (transaction)**, ensuring trust in off-chain resolution logic.

#### [SRL 2 → 3 | Single Service Provider Prototype](https://github.com/input-output-hk/innovation-cavefish/milestone/1)

- **Prototype Development**
  - Progressed on **WBPS wrapper integration** in Haskell and refined circuit flow visualizations via Miro.  
  - Planned a **feasibility spike** to test **on-device verification** using Circom from TypeScript, targeting mobile-friendly execution.  
  - Advanced **Nix shell integration** for reproducible builds and environment portability.  
- **Project Refactoring**
  - Partitioned the main Haskell codebase into **modular sub-projects**, improving build organization and maintainability.  
  - Established a **GitHub Kanban board** and initial issue set aligned with SRL milestones.

#### [SRL 1 | Create Workstream Briefing Pack](https://github.com/input-output-hk/innovation-cavefish/milestone/3)

- Continued production of **non-technical explainers** and **workstream briefs** for community visibility and alignment.

## 📄 Deliverables

### 🧩 Documents
- [CIP #1110 – Intent-Based Transaction Processing (Draft PR)](https://github.com/cardano-foundation/CIPs/pull/1110)  
  Rendered version: [CIP – Intent-Based Transactions (Polina Vinogradova’s branch)](https://github.com/polinavino/CIPs/blob/polina/light/CIP-%3F%3F%3F%3F/README.md)
- Updated **Miro board** describing the WBPS wrapper and **100-ish Days Demo** architecture : (Miro board link: https://miro.com/app/board/uXjVI1nt2yc=/ Password: 1234cacefish) 
<img  alt="image" src="https://github.com/user-attachments/assets/984b659d-1ec6-4b6f-aedd-7308d7243032" />
 
- **[GitHub Kanban](https://github.com/orgs/input-output-hk/projects/188)** initialized to manage issue tracking and milestone mapping.

### 🧩 Commits (October 16 – 31, 2025)

Below is the full list of commits merged into `main` during this period:

* [f48ea69](https://github.com/input-output-hk/innovation-cavefish/commit/f48ea693227167b048dc4fd1fa452ff308e03d35) — Add cardano-node binary to nix shell *(Kayvan ≅ کیوان)*
* [7b71442](https://github.com/input-output-hk/innovation-cavefish/commit/7b7144223a8b257661003d25d2175d8e53455fd8) — Merge pull request #37 from input-output-hk/wg/wip-prototype-server *(Will Gould)*
* [218d3de](https://github.com/input-output-hk/innovation-cavefish/commit/218d3de0c8d979a8cc73d9b4dcb9b027a1dabcbd) — post-merge fixes *(Will Gould)*
* [56fb338](https://github.com/input-output-hk/innovation-cavefish/commit/56fb3382bef301523aa3815e8390dfc868cfec40) — Merge remote-tracking branch 'origin/master' into wg/wip-prototype-server *(Will Gould)*
* [3631736](https://github.com/input-output-hk/innovation-cavefish/commit/3631736a7fbbbe1d5d1c940120fc8c595f53ad00) — add `commit` endpoint *(Will Gould)*
* [84fb8d4](https://github.com/input-output-hk/innovation-cavefish/commit/84fb8d4c06c1eed8d5f857595a700ae7145386af) — Merge pull request #34 from input-output-hk/nhenin/fix-makefile *(Nicolas Henin)*
* [40ec91d](https://github.com/input-output-hk/innovation-cavefish/commit/40ec91d6b6a78525616f37485b1ea8cf0d906c5d) — Merge pull request #33 from input-output-hk/24/cavefish-server-to-sub-projects *(Nicolas Henin)*
* [17e4f24](https://github.com/input-output-hk/innovation-cavefish/commit/17e4f240f62216ec1922c892461001d056a3bbd8) — [wbps] fixed clone issue *(Nicolas Henin)*
* [989b74b](https://github.com/input-output-hk/innovation-cavefish/commit/989b74b4e188b7fa057c093a2ddc4cf4fe3e8355) — [wpbs] improved terms used *(Nicolas Henin)*
* [7934c6d](https://github.com/input-output-hk/innovation-cavefish/commit/7934c6d6c97b5a51209db1bc84e50d659d4a9a25) — Merge pull request #31 from input-output-hk/paper-minor-fix *(Nicolas Henin)*
* [0856d73](https://github.com/input-output-hk/innovation-cavefish/commit/0856d738a2d351942646e455427634b1fa29af89) — Separate the project into sub-project *(Kayvank)*
* [52b05ea](https://github.com/input-output-hk/innovation-cavefish/commit/52b05ea7a32fe684ec4b493cb59e97168bb076e8) — Separate the project into sub-project *(Kayvank)*
* [52cb211](https://github.com/input-output-hk/innovation-cavefish/commit/52cb2118e5b102c6f1d04e271d14e9424d344a88) — Merge pull request #19 from input-output-hk/kb-openSourcing-Oct28 *(Nicolas Henin)*
* [f02eeff](https://github.com/input-output-hk/innovation-cavefish/commit/f02eeff28a308a2d98713833923583968c3f22cb) — Separate the project into sub-project *(Kayvank)*
* [d81cb7e](https://github.com/input-output-hk/innovation-cavefish/commit/d81cb7e1fd18e03ffabfbf6ea85f1cd9f629573d) — Merge pull request #18 from input-output-hk/wg/wip-prototype-server *(Will Gould)*
* [688aac0](https://github.com/input-output-hk/innovation-cavefish/commit/688aac0a9105df047d136bc71cc1b59c7bd8e085) — format *(Will Gould)*
* [5315705](https://github.com/input-output-hk/innovation-cavefish/commit/53157052558de4910fd8a26fdc2775714a9fd016) — Merge remote-tracking branch 'origin/master' into wg/wip-prototype-server *(Will Gould)*
* [a1a4623](https://github.com/input-output-hk/innovation-cavefish/commit/a1a46239beb6dc7deea14b057441f5bb5b1ca268) — Fix typo in docs *(Kayvank)*
* [1e4fc7d](https://github.com/input-output-hk/innovation-cavefish/commit/1e4fc7dea93f05920f06f2be70a2afebddd5f88f) — don't need partial type signatures now, improved error message, placated warnings *(Will Gould)*
* [96c274a](https://github.com/input-output-hk/innovation-cavefish/commit/96c274a49a2d8711511eba87acd3b322fb7aeb96) — added a placeholder for the missing middle endpoint *(Will Gould)*
* [32b495c](https://github.com/input-output-hk/innovation-cavefish/commit/32b495cc9c461c5300058b0a4a049df14e59feca) — Update Readme.md *(Kris Bennett)*
* [b8960dd](https://github.com/input-output-hk/innovation-cavefish/commit/b8960dd608207546cb5e59656ce09906a06aceff) — Update Readme.md *(Kris Bennett)*
* [79bea63](https://github.com/input-output-hk/innovation-cavefish/commit/79bea63a7a3f3286cde2ccf1f46b458d03877fb5) — Create LICENSE.md *(Kris Bennett)*
* [861e5ab](https://github.com/input-output-hk/innovation-cavefish/commit/861e5abe693753ac45fb38e72be4188a7790a38f) — Create NOTICE.md *(Kris Bennett)*
* [95e2d8b](https://github.com/input-output-hk/innovation-cavefish/commit/95e2d8b3b3f27c1ef287d0cdcd4293ac247ddf9a) — Add hlint and formatting *(Kayvan ≅ کیوان)*
* [a302de7](https://github.com/input-output-hk/innovation-cavefish/commit/a302de7c83b09e8f03dab7887d573bc8a0f0e412) — fourmolu formatting, hlint fixes and adding minor docs (#15) *(Kayvan ≅ کیوان)*
* [873185e](https://github.com/input-output-hk/innovation-cavefish/commit/873185e2525987c1b5f6d47da273820c4fb3e7c1) — Merge pull request #14 from input-output-hk/11/fourmolu-formatting *(Nicolas Henin)*
* [57e941a](https://github.com/input-output-hk/innovation-cavefish/commit/57e941a339be66229c2d415564155461ca93c064) — Add github actions *(Kayvank)*
* [53e1ae9](https://github.com/input-output-hk/innovation-cavefish/commit/53e1ae9ac4df0de54cd5d8967e9da3f1b5b7c5a0) — Update .github/workflows/cavefish-server-linux-ci.yml *(Kayvan ≅ کیوان)*
* [db3c2af](https://github.com/input-output-hk/innovation-cavefish/commit/db3c2af89b3f1cf11469b8debe996694e7631995) — Update cavefish-server/nix/shell.nix *(Kayvan ≅ کیوان)*
* [a938c96](https://github.com/input-output-hk/innovation-cavefish/commit/a938c96a57e3477f78f942af1af48ca119676c60) — Add fourmolu config *(Kayvank)*
* [8711f04](https://github.com/input-output-hk/innovation-cavefish/commit/8711f04c3a2631d8dfd9d1ec2e562955e418465a) — Add github action *(Kayvank)*
* [0253daa](https://github.com/input-output-hk/innovation-cavefish/commit/0253daa31199043a01c602dbd5d314d012874707) — Merge pull request #9 from input-output-hk/7/update-docs *(Nicolas Henin)*
* [460d20a](https://github.com/input-output-hk/innovation-cavefish/commit/460d20ae079df701be9035c032b3f67c65548d71) — Update README *(Kayvank)*
* [c46dd85](https://github.com/input-output-hk/innovation-cavefish/commit/c46dd85b269b35fd317593420a2a6d64425fa9f7) — Organize gitignore *(Kayvank)*
* [ab0d2c6](https://github.com/input-output-hk/innovation-cavefish/commit/ab0d2c6aee7ae0ac09af9c41652e84f248497a35) — renaming output files *(Nicolas Henin)*
* [b7eae62](https://github.com/input-output-hk/innovation-cavefish/commit/b7eae62c96c5884b99fc378e2ab8bf7018fbd254) — isolated artefacts related to witness generation *(Nicolas Henin)*
* [0baab1f](https://github.com/input-output-hk/innovation-cavefish/commit/0baab1fa012c8f4bd4eb637a52ac4baadf49e5ce) — Update README *(Kayvank)*
* [9cadda3](https://github.com/input-output-hk/innovation-cavefish/commit/9cadda33a7463b4221b3aa86c922a5a2416db333) — added benchmark report at the end of Makefile *(Nicolas Henin)*
* [3f08a36](https://github.com/input-output-hk/innovation-cavefish/commit/3f08a360880eccaf841b363bf3cee6f680d0b4af) — Merge pull request #3 from input-output-hk/wg/wip-prototype-server *(Nicolas Henin)*
* [9133dac](https://github.com/input-output-hk/innovation-cavefish/commit/9133dac555bcb3f36098d7c1dca2819817050ccc) — removed Prove & Verify (optional / permissive) phased *(Nicolas Henin)*
* [c91d8ab](https://github.com/input-output-hk/innovation-cavefish/commit/c91d8abca9d7846983e50b9ea6e8f6654b645d3f) — improved logs *(Nicolas Henin)*
* [1091c1d](https://github.com/input-output-hk/innovation-cavefish/commit/1091c1d5f82a4bd129e9b0e14dd122bb5c5129cf) — updated CI *(Nicolas Henin)*
* [3a564c4](https://github.com/input-output-hk/innovation-cavefish/commit/3a564c455919ffb5b6880c5e1e5b0be61edee215) — refactored the circuit logic *(Nicolas Henin)*
* [135963e](https://github.com/input-output-hk/innovation-cavefish/commit/135963eecf70c43af2a4c0ae60f098a1fe5bd084) — witness machinery *(Will Gould)*

## 🚀 Next Steps
- Continuing **WBPS wrapper integration** and end-to-end validation flow.  
- Conduct **feasibility spike** for lightweight (on-device) verification.  
- Prepare **UI prototype** and demonstration scenario for the **100-day demo**.  
