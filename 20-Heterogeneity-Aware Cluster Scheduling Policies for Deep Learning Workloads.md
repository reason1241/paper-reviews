# Heterogeneity-Aware Cluster Scheduling Policies for Deep Learning Workloads

Date: 2026-10-17

- [Link](https://www.usenix.org/system/files/osdi20-narayanan_deepak.pdf)

- Problem: Many existing cluster schedulers do not consider the performance heterogeneity of jobs across different accelerator types.
- Solution: The authors propose a heterogeneity-aware scheduler called Gavel, which generalizes many scheduling policies by expressing them as optimization problems using effective throughput. Many of these problems can be solved using linear programming (LP).
- Claimed technical contributions:
  - The authors propose a systematic method to convert scheduling policies into heterogeneity-, colocation-, and placement-aware policies by expressing them as optimization problems.
  - Building on Quasar, Gavel's throughput estimator maps a new job to a similar job from a set of pre-profiled reference jobs to estimate colocated throughputs. Using measured or estimated throughputs for individual jobs and job combinations as inputs, Gavel formulates the optimization problem, solves it to generate a time allocation matrix, and uses this allocation for scheduling.
  - They propose round-based scheduling, using six-minute rounds in their experiments. By managing priorities, Gavel determines which jobs to launch in each round. This helps the allocation received by jobs approach the computed target allocation when individual rounds deviate from it.
  - They show how to formulate optimization problems for makespan, least attained service (LAS), first-in, first-out (FIFO), and other policies.
- Metrics / Baselines:
  - The authors measure average job completion time (JCT), finish-time fairness (FTF), makespan, total cost, and SLO violation rates.
  - They compare Gavel with heterogeneity-agnostic versions of LAS, FTF, and FIFO, Gandiva-style space sharing, and AlloX for single-GPU jobs, using physical and simulated clusters.
  - On the physical cluster, Gavel improves average JCT by 1.5× over LAS and makespan by 1.2× over Gandiva (Table 3). In simulation, it improves average JCT by 3.5× for single-GPU jobs and 2.2× for workloads including multi-GPU jobs over LAS at high load (Figures 9–10).
  - They also evaluate optimization scalability and preemption overhead, compare round-based scheduling with ideal allocation, and compare estimated throughputs with oracle throughputs (Table 4; Figures 14–16).
- Key insight and enabling idea: Many scheduling policies can be expressed as optimization problems using effective throughput—the time-weighted average of a job’s throughput across resource configurations. This allows Gavel to account for heterogeneity, colocation, and placement while supporting different scheduling objectives. A separate round-based mechanism approximates the computed allocations in practice.
- Novelty and impact: Gavel provides a flexible scheduling framework that systematically generalizes existing policies to account for heterogeneity, colocation, and placement. It separates policy optimization from a reusable round-based scheduling mechanism, supporting different objectives without redesigning the scheduler. This improves resource allocation and allows heterogeneous clusters to support higher loads while reducing job completion time, makespan, or cost, depending on the policy.
- Technical Qualities:
  - Framing and assumptions: Gavel targets long-running, iterative DNN training jobs whose throughput can be estimated through short profiling runs. It assumes jobs can be checkpointed and resumed with manageable overhead, enabling time sharing and migration across accelerators.
  - Merits of the technical contributions: Separating policy optimization from scheduling execution allows new policies to reuse the scheduling mechanism by defining suitable objectives and constraints. Effective throughput provides a common abstraction for incorporating heterogeneity, colocation, and placement into different policies.
  - Does the evaluation support claims and reveal limitations of the proposed approach?: Physical-cluster experiments and larger simulations support improvements across several objectives, while comparisons with ideal allocations and oracle throughputs validate the scheduling mechanism and estimator under tested conditions. However, the largest gains come from simulations using generated workloads. Unfamiliar workloads may require additional profiling or produce inaccurate estimates, and optimization costs increase with scale and job combinations. The policies are not generally strategyproof: users may benefit from manipulating throughput information. The evaluation focuses on training; applicability to autoregressive inference serving remains untested.
- Criticism:
  - The largest gains come from simulations using generated workloads.
  - Unfamiliar workloads may require additional profiling or produce inaccurate estimates.
  - Optimization costs increase with scale and job combinations. 
  - The policies are not generally strategyproof: users may benefit from manipulating throughput information. 
  - The evaluation focuses on training; applicability to autoregressive inference serving remains untested.
  - Space sharing is limited to pairs of jobs, and placement is modeled using consolidated and unconsolidated configurations, limiting the configurations the optimizer considers.
  - The evaluation provides limited evidence for workloads whose throughput changes substantially during execution, where earlier profiling measurements may become less representative.
