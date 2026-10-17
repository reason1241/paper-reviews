# DistServe: Disaggregating Prefill and Decoding for Goodput-optimized Large Language Model Serving

Date: 2026-10-17

- [Link](https://arxiv.org/abs/2401.09670)

- Problem: Existing LLM serving systems colocate the prefill and decoding phases and batch the computation of prefill and decoding, which causes interference and couples their resource allocation and parallelism strategies, making it difficult to achieve both low time to first token (TTFT) and low time per output token (TPOT) cost-efficiently.
- Solution: The authors propose an LLM serving system called DistServe, which disaggregates the prefill and decoding computation.
- Claimed technical contributions:
  - The prefill phase is typically compute-intensive, while decoding with small batches is memory-bandwidth-bound. By disaggregating the two phases, DistServe can optimize them differently.
  - DistServe proposes placement algorithms for clusters with high-bandwidth or low-bandwidth connections between nodes. A simulator plays a key role in predicting per-GPU goodput, which is the maximum request rate that meets the required SLO attainment target, divided by the number of GPUs.
  - The system reduces pipeline bubbles by batching short prefills toward a profiled token threshold and scheduling longer prefills individually. It also dispatches requests to the prefill instance with the shortest queue and then to the least loaded decoding instance for decoding steps.
  - DistServe profiles the workload pattern by monitoring the average input and output lengths and the average arrival rate. It replans the placement when it detects a significant change in the workload pattern.
- Metrics / Baselines:
  - The authors measure SLO attainment under TTFT and TPOT constraints, maximum per-GPU goodput at 90% attainment, and the tightest latency targets supported at a fixed request rate.
  - They compare DistServe with vLLM and DeepSpeed-MII using OPT models and ShareGPT, HumanEval, and LongBench workloads with synthetic Poisson arrivals.
  - DistServe supports up to 7.4× higher per-GPU request rates than DeepSpeed-MII or 12.6× tighter latency targets than vLLM in different tested settings (Figures 8–9).
  - Simulation ablations compare the two placement algorithms with vLLM and vLLM++, which searches alternative parallelism configurations (Figure 11).
- Key insight and enabling idea: Prefill and decoding have different computational characteristics and latency requirements. Disaggregating them removes their execution interference on shared GPUs and enables different resource allocation and parallelism strategies for each phase. Bandwidth-aware placement exploits fast intra-node links for KV-cache transfers when cross-node bandwidth is limited, making disaggregation practical.
- Novelty and impact: DistServe combines prefill–decoding disaggregation with phase-specific resource allocation, parallelism optimization, and bandwidth-aware placement to maximize per-GPU goodput under TTFT and TPOT constraints. This allows more requests to meet latency requirements with fewer GPU resources.
- Technical Qualities:
  - Framing and assumptions: Prefill is typically compute-bound, while decoding with small batches is memory-bandwidth-bound. The network must support both timely individual KV-cache transfers and aggregate traffic. DistServe exploits the bandwidth hierarchy between intra-node and inter-node connections. Its simplified queueing analysis assumes constant execution time, while the high-bandwidth placement algorithm assumes negligible cache-transfer overhead.
  - Merits of the technical contributions: Disaggregation isolates prefill and decoding execution and enables different resource allocation and parallelism strategies for each phase. Bandwidth-aware placement reduces cache-transfer overhead, while workload-based simulation guides configuration selection.
  - Does the evaluation support claims and reveal limitations of the proposed approach?: Experiments across multiple models and workloads support improved per-GPU goodput under TTFT and TPOT constraints and show small cache-transfer overhead in the tested placements. However, the broader placement ablation relies on simulation, although simulator accuracy is checked against real runs. The experiments do not establish the same benefits for models beyond 175B parameters or much longer contexts, where cache-transfer costs and memory occupied by additional model copies may become more significant.
- Criticism:
  - The broader placement ablation relies on simulation, although simulator accuracy is checked against real vLLM and DistServe-Low runs.
  - The experiments do not establish the same benefits for models beyond 175B parameters or much longer contexts. Larger models increase the memory required for additional model copies, while longer contexts increase KV-cache memory and transfer requirements.
  - Workloads use synthetic Poisson arrivals, limiting evidence for correlated bursts and changing traffic patterns, despite the runtime mechanisms intended to handle them.
  - The evaluation does not directly compare DistServe with other disaggregated systems, such as Splitwise, leaving its performance relative to alternative disaggregation designs unestablished.
  - Advanced preemption and fault tolerance remain future work. Long requests can block shorter requests under FCFS scheduling, while dependencies between prefill and decoding instances introduce failure-propagation risks.
