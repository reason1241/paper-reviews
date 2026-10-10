# Orca: A Distributed Serving System for Transformer-Based Generative Models

Date: 2026-10-10

- [Link](https://www.usenix.org/conference/osdi22/presentation/yu)

- Problem: Transformer-based generative models generate tokens in an autoregressive manner, while existing inference serving systems are inefficient for this multi-iteration characteristic because request-level scheduling prevents early-finished requests from returning immediately and late-joining requests from entering the ongoing batch.
- Solution: They propose a distributed serving system called Orca. This system adopts iteration-level scheduling to handle early-finished and late-joining requests and selective batching to batch an arbitrary set of requests.
- Claimed technical contributions:
  - Batching in existing systems wastes computation since the engine continues processing a batch until all requests finish, even though some requests have already finished. Iteration-level scheduling solves the problem by selecting requests to run next, invoking the execution engine to execute one iteration, and receiving results for the scheduled iteration. Therefore, finished requests can return immediately, and later requests can run without waiting for the entire batch to finish.
  - In order to achieve high throughput, batching is necessary. However, conventional batching cannot directly combine requests that have different numbers of input tokens, token indices, or phases. Selective batching solves this issue by packing tensors into a single large tensor for non-Attention operations, including QKV computation, splitting it by request before Attention, computing attention separately for each request, and merging the attention outputs.
  - The authors implemented the ORCA system, which includes the scheduler and the execution engine. The execution engine consists of an engine master and worker processes, each containing a controller and GPU-controlling CPU threads. The system supports intra-layer and inter-layer parallelism and pipelines execution across batches. It also reserves KV memory for each admitted request’s maximum sequence length and separates control messages from GPU tensor transfers to reduce unnecessary CPU–GPU synchronization.
- Metrics / Baselines:
  - Metrics include batch execution time, throughput (requests/s), and median end-to-end latency normalized by generated-token count (ms/token). Homogeneous-request experiments report unnormalized median latency.
  - The main baseline is NVIDIA FasterTransformer, with a custom request-level scheduler for end-to-end experiments.
  - Engine microbenchmarks use fixed homogeneous batches with the Orca scheduler disabled. End-to-end experiments use synthetic requests arriving over time, with varying or homogeneous token lengths.
  - For the 175B model, Orca provides a reported 36.9× throughput improvement at a matched median normalized latency of 190 ms/token.
- Key insight and enabling idea: The key idea is to process as many ready tokens as possible per model-parameter read. Non-Attention operations act independently on each token, allowing requests at different stages to share batched computation and reuse loaded parameters, while Attention preserves each request’s own history.
- Novelty and impact: Orca combines iteration-level scheduling with selective batching to efficiently process Transformer requests at different token positions and phases. Its integration of the scheduler and execution engine allows batch membership to change between iterations. For the evaluated 175B workload, Orca achieves a reported 36.9× throughput improvement over FasterTransformer at the same median normalized latency, demonstrating the benefits of reducing waiting and wasted computation.
- Technical Qualities:
  - Framing and assumptions: The paper frames request-level scheduling as inefficient for autoregressive Transformer inference because requests have different arrival times and token lengths. It assumes that request-specific KV states persist across iterations and that each request specifies a maximum total token count for memory reservation.
  - Merits of the technical contributions: Iteration-level scheduling reduces waiting and wasted computation, while selective batching preserves shared-parameter reuse despite different request states. KV-memory reservation prevents capacity-related deadlock, and distributed pipelining and control/data-plane separation improve execution efficiency for large models.
  - Does the evaluation support claims and reveal limitations of the proposed approach?: Engine microbenchmarks and end-to-end experiments support the claimed improvements under the tested conditions. They also reveal small engine overheads for the 13B and 101B models and weaker performance when batching is limited. However, the workloads are synthetic, output lengths are prescribed with EOS disabled, and median latency does not characterize tail latency. The potential costs of very long prompts and conservative KV-memory reservation are not thoroughly evaluated.
- Criticism:
  - The workloads are synthetic, which may not reflect the real workloads.
  - Median latency does not characterize tail latency. 
  - The potential costs of very long prompts and conservative KV-memory reservation are not thoroughly evaluated.
  - The evaluation does not fully isolate how much scheduling, memory-management, and communication optimizations individually contribute to the end-to-end speedup.
  - The scheduler stops selecting requests when an earlier request’s KV reservation cannot fit, even if later requests require less memory. This may leave available capacity unused while smaller requests wait.
  - The evaluation does not report time to first token or intervals between generated tokens, leaving interactive generation latency unclear.
