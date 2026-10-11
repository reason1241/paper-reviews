# Efficient Memory Management for Large Language  Model Serving with PagedAttention

Date: 2026-10-11

- [Link](https://dl.acm.org/doi/10.1145/3600006.3613165)

- Problem: Existing systems struggle to achieve high throughput because each request’s KV cache is large and grows and shrinks dynamically; memory reservation, fragmentation, and redundant duplication waste memory and limit batch size.
- Solution: The authors propose PagedAttention, an attention algorithm inspired by virtual memory and paging that stores the KV cache in non-contiguous blocks and enables sharing, and build vLLM to reduce memory waste and improve serving throughput.
- Claimed technical contributions:
  - The authors introduce PagedAttention, which computes attention over KV cache stored in non-contiguous, fixed-size blocks. vLLM allocates the block pool in advance and assigns blocks to requests on demand.
  - The KV cache manager manages logical KV blocks and physical KV blocks using block tables. These block tables map logical blocks to physical blocks and store the number of filled positions.
  - They implement fused reshape and block write, block read and attention, and block copy kernels to support paged memory access and reduce overhead.
  - They enable KV cache sharing within and across requests using reference counting and block-level copy-on-write, supporting parallel sampling, beam search, and shared prefixes.
  - They handle growing KV caches through first-come-first-serve scheduling, sequence-group preemption, and recovery through swapping or recomputation.
- Metrics / Baselines:
  - The main metric is normalized latency, the mean of each request’s end-to-end latency divided by its output length (s/token). They compare sustainable request rates at similar normalized latency and also measure average batch size, KV cache memory savings, and kernel latency.
  - The baselines are FasterTransformer with a custom dynamic-batching scheduler and a reimplementation of Orca assuming buddy allocation. Orca (Max) reserves the maximum sequence length; Orca (Pow2) rounds the known output length up to a power of two; Orca (Oracle) reserves according to the known output length.
  - They evaluate OPT-13B, OPT-66B, and OPT-175B using ShareGPT- and Alpaca-derived workloads with synthetic Poisson arrivals, and LLaMA-13B using WMT16-derived shared-prefix workloads, on A100 GPUs.
  - For basic sampling on ShareGPT, vLLM sustains 1.7–2.7× higher request rates than Orca (Oracle) and 2.7–8× higher than Orca (Max) at similar normalized latency (Figure 12).
- Key insight and enabling idea: OS style paging separates logical KV blocks from physical KV blocks, allowing on-demand allocation in non-contiguous memory and sharing of identical prefixes. PagedAttention enables attention over these blocks, reducing memory reservation, fragmentation, and redundant duplication so more requests fit into a batch, improving serving throughput.
- Novelty and impact: Unlike existing serving systems that store each request’s KV cache contiguously, vLLM adapts OS-style paging to KV cache management through PagedAttention, enabling on-demand block allocation and flexible sharing.
- Technical Qualities:
  - Framing and assumptions: Requests arrive at different times and have variable input and unknown output lengths, making batching and KV cache allocation challenging. The paper targets memory-bound serving, where reducing memory waste allows larger batches and higher throughput.
  - Merits of the technical contributions: PagedAttention reduces memory reservation and fragmentation, while reference counting and copy-on-write enable KV cache sharing. These mechanisms allow larger batches without requiring each sequence’s cache to be physically contiguous.
  - Does the evaluation support claims and reveal limitations of the proposed approach?: The evaluation supports improved throughput at similar normalized latency across the tested workloads and demonstrates memory savings from sharing. However, Orca is reimplemented with assumed allocation behavior, and arrivals are synthetically generated. The evaluation also reveals paging overhead and smaller benefits in compute-bound settings, so the improvements should not be generalized to all workloads.
- Criticism:
  - Orca is reimplemented with assumed allocation behavior, and arrivals are synthetically generated, limiting how closely the evaluation represents the original baseline and real serving traffic.
  - The evaluation reveals paging overhead and smaller benefits in compute-bound settings, so the improvements should not be generalized to all workloads.
  - Performance across broader input/output length distributions is not systematically evaluated, particularly long-input, short-output workloads such as summarizing multiple articles.
  - The main metric is mean normalized latency, which does not directly capture tail latency, time to first token, or inter-token latency.
