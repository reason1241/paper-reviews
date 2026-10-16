# Taming Throughput-Latency Tradeoff in LLM Inference with Sarathi-Serve

Date: 2026-10-16

- [Link](https://www.usenix.org/conference/osdi24/presentation/agrawal)

- Problem: Existing LLM serving systems achieve higher throughput using batching, but they struggle to achieve both high throughput and low latency due to interference between prefill and decoding.
- Solution: The authors propose an LLM serving system called Sarathi-Serve, which utilizes chuncked prefills and stall-free schedules.
- Claimed technical contributions:
  - The authors identify limitations in existing scheduling policies, such as vLLM’s and Orca’s prefill-prioritizing scheduling and FasterTransformer’s decode-prioritizing scheduling, which trade off throughput and token-generation latency. They also show that memory-bound decode batches underutilize GPU compute.
  - The prefill phase usually takes longer than a decode iteration. Including an entire prefill and ongoing decodes in one iteration worsens time-between-tokens (TBT) latency because the iteration takes longer. The authors propose chunked prefills, which split prefill computation into smaller pieces processed across multiple iterations.
  - For stall-free scheduling, the system first determines a token budget based on the TBT SLO and allocates batch space in the order of ongoing decodes, already admitted unfinished prefills, and new prefills. By combining decode prioritization with chunked prefills within this budget, the system reduces generation stalls and improves TBT SLO attainment. This approach also reduces pipeline bubbles—GPU idle time caused by large variations in microbatch runtimes depending on their composition of prefill and decode tokens—by creating batches with more uniform computational requirements.
- Metrics / Baselines:
  - The authors measure median time to first token (TTFT), P99 time between tokens (TBT), and serving capacity—the maximum sustainable request rate under latency constraints. They also measure chunking overhead relative to unchunked prefill.
  - They compare Sarathi-Serve with vLLM and Orca across four models using conversation and document-summarization workloads under strict and relaxed TBT SLOs.
  - Under strict SLOs on the conversation workload, Yi-34B achieves up to 3.7× the capacity of vLLM and 4.0× that of Orca (Figure 10). Falcon-180B achieves 3.6× the capacity of vLLM with the same tensor- and pipeline-parallel configuration (Figure 13).
  - Ablations compare hybrid batching alone, chunked prefills alone, and their combination (Table 4), while Figure 14 evaluates chunking overhead.
- Key insight and enabling idea: Memory-bound decode batches leave spare GPU compute capacity that can process prefill work. Prefills can be split into chunks while preserving context through the KV cache. Combining these chunks with ongoing decodes within a token budget balances throughput and TBT latency, while more uniform computation per iteration reduces pipeline bubbles.
- Novelty and impact: Building on earlier SARATHI work, Sarathi-Serve combines chunked prefills with stall-free scheduling and an SLO-aware token budget to control prefill–decode interference. This increases sustainable serving capacity under TBT constraints and reduces pipeline bubbles, allowing more requests to be served efficiently on the same GPU resources.
- Technical Qualities:
  - Framing and assumptions: The approach targets online LLM serving, where prefill is typically compute bound and small decode batches are memory bound. It exploits spare compute capacity in decode batches by adding prefill chunks. Both TTFT and TBT matter, while token-budget selection depends on the model, hardware, parallelism configuration, and TBT SLO.
  - Merits of the technical contributions: Chunking and stall-free batching reduce prefill interference with ongoing decodes and improve GPU utilization. More uniform microbatch runtimes also reduce pipeline bubbles. These benefits increase serving capacity despite the additional prefill overhead introduced by chunking.
  - Does the evaluation support claims and reveal limitations of the proposed approach?: Experiments across multiple models and deployments support the capacity improvements, while ablations and overhead measurements expose tradeoffs. However, TTFT results are limited to median values in a small ablation, leaving tail TTFT and joint TTFT–TBT constraints insufficiently evaluated. Workloads use Poisson arrivals, limiting evidence for correlated or time-varying traffic bursts. Token budgets are fixed per configuration, and no quantitative comparison with disaggregated systems such as DistServe is provided.
- Criticism:
  - TTFT results are limited to median values in a small ablation, leaving tail TTFT and joint TTFT–TBT constraints insufficiently evaluated.
  - Workloads use Poisson arrivals, limiting evidence for correlated or time-varying traffic bursts.
  - Token budgets are fixed per configuration, and no quantitative comparison with disaggregated systems such as DistServe is provided.
  - No direct comparison with earlier SARATHI isolates the incremental benefits of the revised scheduler beyond the existing chunked-prefill and hybrid-batching mechanisms.
  - Pipeline-parallel evaluations use two stages, leaving the benefits and scheduling behavior of deeper pipelines unestablished.
