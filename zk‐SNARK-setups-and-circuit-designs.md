# Setup phases in zk-SNARKs

All zk-SNARKs require some sort of public parameters to be generated first. However, there are different type of setups that can be classified according to two dimensions:

* **Transparent vs. Trusted:** determines if the public parameters of the setup must be derived from some private parameters (trusted) or if they can be directly sampled from some distribution (transparent), without private data involved. The private parameters are commonly referred to as toxic waste. If the private parameters are exposed, security is broken (it normally allows a malicious prover to forge fake proofs). The part of the setup that involves the private parameters can be executed by a single Trusted Third Party or, commonly in practice, by a set of distrustful parties running a Secure Multi-Party Computation (MPC) process, where none of the parties learns the full private value.
* **Universal vs. Circuit-Specific:** determines if the public parameters from a setup can be reused to generate proofs on different circuits or, on the contrary, a special circuit-specific setup phase must be run independently for each circuit. Note that, even when a universal setup can be reused for different circuits, some SNARKs still require to derive a proving/verifying key pair from each specific circuit to be used.

From a usability point of view, protocols with transparent and universal setups are the most versatile ones because they enable an easier handling of different circuits without re-runing setup steps or relying on private parameters. However, this normally comes at the cost of having slower proof systems. Some illustrative examples are presented below, with asymptotic costs relative to the circuit size:

| **Protocol** | **Setup** | **Prover time** | **Verifier time** | **Proof size** |
| ------------ | --------- | --------------- | ----------------- | -------------- |
| Groth16 | Trusted / Circuit-Specific | linear | linear (w.r.t. public statement x) | constant |
| Halo2 (KZG) | Trusted / Universal | quasi-linear | linear (w.r.t. public statement x) | constant |
| Halo2 (IPA) | Transparent / Universal | linear | logarithmic | logarithmic |
| Bulletproofs | Transparent / Universal | quasi-linear | linear | logarithmic |

A high-level representation of how different types of setups are used is presented in the following diagram.

<img width="781" height="461" alt="Setups" src="https://github.com/user-attachments/assets/6f499dd3-2e15-4b24-9757-dab10b41f585" />

# Circuit difficulties in Cavefish

Cavefish presents a set of issues that must be resolved in order to achieve efficient resolution of transactions. 

The main bottleneck in performance is the zk-SNARK because of the computational costs involved in generating the setup/key material (if needed) and the proof. Depending on the amount of checks to be performed inside the circuit, this can become quite large. 

In addition, Cavefish rely on some ZK UNfriendly primitives that cannot be modified because they replicate Cardano native operations, e.g., Blake2b to compute $txId$ from the transaction body and Sha512 to compute the EdDSA challenge. These hash functions are not very efficient inside circuits unless a big effort is done in optimizations (and these are very proof system-dependent), a problem that is exacerbated by the fact that the main private input of the circuit, i.e., the transaction body, can present different lengths that depend on the fields being used to satisfy each intent (such as the number of UTxO inputs). In addition, some transactions can be quite large to be handled efficiently inside circuits, mainly when computing the hash.

# Arithmetic circuits with variable length inputs

zk-SNARKs rely on arithmetic circuits that must be fully evaluated in a deterministic way. This implies a set of considerations that really affect usability and performance, overall in solutions such as those from the privacy ecosystem which already are inefficient for some use cases. For example:

* Modifying the length of the input implies a modification in the circuit structure, directly in the number of wires for the public input and witness, and indirectly in the amount of computation (e.g., number of rounds or size of internal structures) that is needed by the functions used inside the circuit.
* There is no efficient conditional branching. Since the circuit evaluation is deterministic, all branches have to be evaluated and the desired result must be filtered out based on some arithmetic trick using selectors. This is more complex to handle when the selected branch must be kept private, because the implementation can leak information about this if not implemented correctly.

## The impact of setup phases

In the case of zk-SNARKs, modifying the circuit structure may imply quite a large restriction in usability because of the need of setups, mostly when they are trusted. Proof systems such as Groth16 achieve efficient and compact proofs, but requires two setup phases with toxic waste which are slow if centralized and overall if run by some MPC. Therefore, modifying the circuit on demand is not realistic in this case. In other systems such as Plonk, using a universal setup makes this easier to handle, but there is still a need of computing the prover/verifier keys for each specific circuit. This last step can still be inconvenient in systems where many different circuits are needed or there is not time enough for computing the keys each time (or having not enough resources available).

## General strategies to handle variable length inputs

Handling variable length inputs inside arithmetic circuits is complex and typically introduce quite large overheads. From a general perspective, there should be three main solutions:

1. Forcing the input length to be used to a specific invariant length and using padding strategies if required, and where the circuit always computes the same amount of steps. This option is not very realistic, mainly when the zk-SNARK is replicating an external computation that cannot be updated to a padded scenario.
2. Fixing fine grained cases for different lengths and having a specific circuit to be used for each one of the cases. This can be useful when the set of possibilities is not too big such that the system can rely on having $N$ circuits with their respective independent setups and prover/verifier key pairs. However, if the variability of the input lengths is large, $N$ can raise in an unmanageable way.
3. Using a buffered approach. In this case, a single circuit is designed to take as private input a single buffer up to a length $L_{max}$. All inputs that are smaller can be provided with the corresponding padding. However, instead of fixing the computation to process the whole input buffer, the circuit is also provided a private input that captures the real length of the input. This allows the circuit to only process the useful amount of data inside the buffer and obtain the desired results. However, due to different caveats, this approach typically implies some overhead in processing the dummy data too (even if the circuit manages to discard the unwanted results).

From the three cases, (3) is the more realistic one for most of the applications. However, the most large $L_{max}$ is, the most complexity (and potential overhead) the circuit will have. Therefore, it should not be discarded the option to combine (2) and (3) in a way that a nice trade-off is found between the number of circuits to be used and the complexity of each of them. One could select, for example, to use 3 circuits such that:

1. Circuit 1 handles inputs of length $(0, L)$
2. Circuit 2 handles inputs of length $(L, 2L)$
3. Circuit 3 handles inputs of length $(2L, L_{max})$

### The special case of hashing

Hash functions typically rely on some sort of chunk solution, where data is padded until a multiple of the chunk length $C_L$ is achieved. Then, the number of chunks $N$ determine the number of rounds to be executed.

To work with variable sized data, the circuit must handle an input buffer $B$ with a fixed number of chunks $N_{max}$, where the desired data of length $l$ is inserted in $B[0:l]$ and the rest of the buffer is filled with dummy data. This implies that the circuit can only process data of length less or equal than $C_L \cdot N_{max}$ (this limit is a key design constraint) and that all chunks in $B$ will be processed by the hash block, otherwise a modification in the circuit structure would be implied.

To process the desired hash result, the circuit must be provided with another input that captures the real data length $l$ (can be private or public depending on the use case). Then, based on $l$, the circuit will discriminate which chunks are actually contributing to the final result and which have to be discarded. For this, there are two main strategies, depending on the use of the hash function gadget as a black box (less efficient) or a specific design for this purpose (more efficient).

#### Hash function as a black box

For this strategy, it is important to notice that, if the hash function needs to process $i$ chunks, it requires to sequentially update the state at least $i$ times (let’s call this $H_i$). 

*Hashing phase*

This strategy implies to execute the hash function $N$ times, each time considering one additional chunk of the input, i.e., $(H_1, H_2, …, H_N)$. Notice that $chunk_i$ will be processed $i$ times, so there will be a large waste of resources. Then, each hash block will produce a result $r_i = H_i(chunk_1 || … || chunk_i)$.

*Conditional selector phase*

The final step is selecting the desired result $r_i$ from the set of all $N$ results obtained. To do so, the standard arithmetic trick is to use a binary selector $s$ of length $N$ and where only 1 digit is set to 1 (this must also be checked inside the circuit). Then, the desired result is computed as $\sum_i s_i * r_i$, where all but one will be filtered out.

*Cost*

Let’s assume that the hash function cost is dominated by a $compress$ function that works on each chunk at a time. Since the step $H_i$ implies compressing the first $i$ chunks, we have the following cost:
$compress(chunk_1) * N + compress(chunk_2) * (N-1) + … = (N*(N+1))/2 * compress = O(N^2)$

#### Single hash with conditional state updating

This strategy implies to work at the internal hash function level. The idea is to sequentially compress all chunks and only contribute to the final state if enabled by the binary selector. Then, for each chunk $i$:

- Compute state $st_i = compress(st_{i-1}, chunk_i)$
- Output the new state if the chunk is included, otherwise output the previous state, i.e, $st_{out} = st_{i-1} + s_i * st_i$.

*Cost*

Now, there is a sequential execution of the hash function internals (the compression) and therefore the cost is $O(N)$.


