# Recommendations for future steps

Cavefish faces three major difficulties that need to be addressed in future versions for its practical feasibility, mainly regarding the zk-SNARK. The main reasons are the large running times needed to compute the proof and the inflexibility provided by arithmetic circuits to handle variability and optional scenarios. Therefore, three specific parts that should be addressed in parallel are identified:

- **Improve prover performance.** Reduce the cost (in terms of running time / equipment needed) to produce the zk-SNARK. Note that the main bottlenecks are the hash functions SHA2 and BLAKE2.
- **Improve flexibility.** Handle variable length inputs (i.e., Cardano transactions). This requires some additional machinery in the proof system frontend, such as VarLen hashing, conditional updaters, etc. There is few work available for this and some development time should be required.
- **Add selective disclosure.** To move from a full disclosure setting (e.g., the light intent case, where only the inputs are private) to a selective disclosure setting (i.e., only certain fields are exposed to the LC), some additional mechanisms are required to correctly very that the exposed fields are correctly placed in the private transaction to be signed. This will probably imply a solution that handles a CBOR parser or any other type of structure verification inside the SNARK, solutions that are neither trivial nor currently available in existing libraries. In fact, parsing inside SNARKs is an active area of research [[MHN+24](https://eprint.iacr.org/2024/562), [ACM+25](https://eprint.iacr.org/2025/1420)].

## Improve prover performance (hashing)

Binary-oriented hashing is very costly in zk-SNARKs and that’s the main reason that SNARK-friendly hashes such as Poseidon have become so attractive. However, for certain applications (such as Cavefish), the hash used in the infrastructure can’t be modified. Cavefish requires to hash the full transaction in order to compute its txId, and this can imply quite large running times for the prover, mostly with large-sized transaction. This section provides a roadmap to test the potential solutions for reducing the prover time by following a cost-effective strategy. It must be noted that running times provided below are for specific instances (one BLAKE2 preimage, etc.) and that additional components must be taken into consideration inside the SNARK.

### 1️⃣ Use zkVMs

In this case, zkVMs are considered because they can be benefited from applications optimized for binary-oriented operations, such as SHA2/BLAKE2. Risc Zero, for example, compiles Rust programs into Risc-V instructions, which are then fed to the prover. This also allows to optimize the application at the Risc-V level, as they have already done with SHA2. In fact, this benefit of working with binary-oriented architectures can also benefit additional checks to be done to the structure and fields of a transaction, if needed by the specific version of Cavefish.

This is set as the first step in the roadmap because developing the application to be proven is quite fast (e.g., Rust, without the need to face low-level DSLs for arithmetic circuits). Therefore, testing performance in different setups should not take a lot of time and would help in determining if this is an acceptable path to adopt.

However, there are some difficulties expected regarding zkVMs:

- Increasing performance for byte-oriented operations typically implies loosing performance in algebraic ones. In the case of Cavefish, this would be affected by the commitment, which relies on EC-ElGamal.
- Proofs tend to be quite large and adding steps of recursion (for succinctness) can impact the prover time significantly. Thankfully, Cavefish does not require the proofs to be on-chain, therefore large proofs (at a certain extent) might be acceptable.
- zkVMs provers tend to be costly because they target general-purpose applications. Because of this, they can scale quite well with computational resources, but this might be far from what it is expected for SPs.
- Specific optimizations needed might not exist yet (e.g., for BLAKE2) and developing them at the VM-level can be time consuming.

**Current evidence on performance**

The resource [[zk-bench](https://zkbench.dev/)] provides some results for different hash functions in different zk-VMs. It can be seen that Risc Zero is the most optimised for SHA2, providing good performance (< 10s) even with large inputs (10kB) in a 16x CPU setting. The other zk-VMs work in different orders of magnitude.

In [[zk-benchmark](https://github.com/delendum-xyz/zk-benchmarking?tab=readme-ov-file#prover-performance)] there are also SHA2 prover times for both Risc Zero and MidenVM. However, the inputs are only given for sizes of 10 and 100 bytes. In this case, Risc Zero can beat the < 10s target with an Apple M2 CPU. MidenVM also provides times for Blake3, which should be expected faster than Blake2.

There are also benchmarks for different applications in different setups provided by Risc Zero [[r0-benchmarks](https://reports.risczero.com/release-3.0/applications-benchmarks/macOS-apple_m2_pro)], although they seem to be quite slow for the target in Cavefish.

### 2️⃣ Unequal hashing

The work proposed in [[KVW25](https://eprint.iacr.org/2025/1500.pdf)] provides a clean solution to this problem, since it allows to always hash with Poseidon inside the SNARK, with the additional cost of a Universal Hash Function instance per preimage to be hashed. The UHF is modeled as a polynomial of degree `d` which equals the length of the input (encoded as field elements). The good point is that evaluating the polynomial takes only `d` constraints when using Horner’s method, which makes it more suitable for SNARKs. In addition, another benefit for this solution is that it is proof system agnostic (although it targets algebraic settings) and therefore more versatile, contrary to solutions 1️⃣ and 3️⃣ that target specific proof systems.

However, it must be taken into account that this work is quite novel, still not peer reviewed and that it also makes some (common) security assumptions such as the Random Oracle Model.

There is also an important security bound for the size of the field used in the SNARK, which is related to the length of the input: `|F| >= k * 2^(2*lambda)`. Fortunately, if we set the maximum size of a Cardano transaction to be around `16 KiB`, that would imply around `k=518` field elements in `F^254` (e.g., BN254) and a security level of `lambda~=122 bits`, which can be acceptable. Note that security could be improved by using a larger field, e.g., with BLS-318.

**Current evidence on performance**

The following running times are estimated from a linear regression based on the data provided in the paper, which is obtained with a setup of a 10-core Macbook running MacOS 15.6 (kernel 24.6.0), with Apple M1 Max and 32GB of memory.

| **Tx size (B)** | **Prover time (s)** |
| --- | --- |
| 84 `(tx_min)` | 1.20 |
| 127 `(paper)` | 1.25 |
| 494 `(tx_p50)` | 1.67 |
| 508 `(paper)` | 1.69 |
| 785 `(tx_avg)` | 2.00 |
| 906 `(tx_p75)` | 2.14 |
| 1326 `(tx_p90)` | 2.62 |
| 2512 `(tx_p95)` | 3.97 |
| 5983 `(tx_p99)` | 7.91 |
| 8128 `(paper)` | 10.35 |
| 15036 `(tx_max)` | 18.20 |

### 3️⃣ Folding

Folding is a technique that allows to fold multiple instances into a single one (this can be thought of as a precomputation step, cheaper than producing a SNARK) and later prove only the folded one inside a SNARK. For the case of Cavefish, since we target Merkle-Damgard hashing structures, it can be used in its simplest form, i.e., IVC, where a single instance is folded into an accumulated instance each time. Therefore, each instance will capture one application of the compression function to the hash state.

Despite folding is a powerful technique that certainly improves on both prover time and proof size, schemes are quite novel, many still in research stages. Fortunately, there are some implementations [[Nova](https://github.com/microsoft/Nova), [Sonobe](https://github.com/privacy-ethereum/sonobe) (Nova, HyperNova, ProtoGalaxy), [Midnight-ZK](https://github.com/midnightntwrk/midnight-zk/tree/iquerejeta/protogalaxy/proofs/src/protogalaxy) (one-shot ProtoGalaxy)] that allows us to approximate some rough times. In general, to hash `n` data blocks, one could roughly approximate the prover time by `n*t_f+t_p`, where `t_f` is the time for folding one additional instance (of specific size) and `t_p` is the time for producing a valid SNARK on a folded instance (of the same specific size). However, complexities such as the rigidity of current libraries, lack of tools to embed multiple functionalities or the need to work at low-level DSLs to specify the arithmetic circuits must be taken into account. All these make folding as a solution with great potential but in the long term.

**Current evidence on performance**

*Setup:* WSL, i7-13620H 10-cores (2.40 GHz), 16 GB RAM

- **Benchmark: SHA256 in Nova** [`n*t_f`] (Only folding steps)

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

- **Benchmark: Compressed SNARK in Nova** [`t_p`] (Executed once at the end)

| **Num. of constraints in step circuit** | **Prover time** (s) |
| --- | --- |
| 0 | 2.15 |
| 6399 | 3.04 |
| 22783 (SHA256) | 3.45 |
| 55551 | 4.39 |
| 121087 | 6.32 |
| 252159 | 10.33 |
| 514303 | 16.83 |
| 1038591 | 35.10 |

- **Benchmark: Sonobe** [`t_f`] (A single folding step)

| **Num. constr.** | **Nova - Pt (s)** | **Vt (s)** | **HyperNova - Pt (s)** | **Vt (s)** | **ProtoGalaxy - Pt (s)** | **Vt (s)** |
| --- | --- | --- | --- | --- | --- | --- |
| 2^0 | 0.319 | 0.053 | 0.371 | 0.066 | 0.273 | 0.025 |
| 2^14 | 0.411 | 0.070 | 0.498 | 0.091 | 0.316 | 0.061 |
| 2^16 | 0.674 | 0.082 | 0.674 | 0.102 | 0.488 | 0.086 |
| 2^18 | 1.853 | 0.271 | 1.877 | 0.258 | 1.255 | 0.242 |
| 2^19 | 3.561 | 0.544 | 3.741 | 0.505 | 2.228 | 0.404 |
| 2^20 | 7.605 | 0.961 | 8.118 | 0.891 | 4.318 | 0.862 |