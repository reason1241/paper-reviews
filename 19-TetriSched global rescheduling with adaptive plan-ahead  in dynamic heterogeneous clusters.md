# TetriSched: global rescheduling with adaptive plan-ahead  in dynamic heterogeneous clusters

Date: 2026-10-05

- [Link](https://dl.acm.org/doi/10.1145/2901318.2901355)

- Problem: Large clusters serve a mix of jobs with time-based service level objectives (SLOs) and best-effort (BE) jobs. In Hadoop YARN, these jobs are scheduled by a runtime scheduler coupled with a time-based resource reservation system. However, the runtime scheduler enforces a static reservation plan and is unable to leverage resource-type or start-time flexibility.
- Solution: The authors propose a scheduler called TetriSched, which uses estimated runtimes, job deadlines, and placement preferences to plan ahead and globally re-evaluate the schedule for all pending jobs on each scheduling cycle.
- Claimed technical contributions: 
  - This paper introduces Space-Time Request Language (STRL). Its main primitive, nCk, specifies an equivalence set, the number of resources selected from the set, start time, estimated duration, and value associated with the allocation. A mixed-integer linear programming (MILP) objective and constraints are built from these expressions and solved for the scheduling problem. By considering all pending jobs together, TetriSched globally optimizes their placements and start times.
  - TetriSched re-evaluates the schedule on every scheduling cycle for all pending jobs, keeping future allocation plans provisional. This allows more efficient resource utilization by allocating jobs to slots freed by early completion and improves SLO attainment by adapting pending-job schedules to new arrivals and runtime estimation errors.
  - TetriSched handles heterogeneous resources by using different equivalence sets to represent job-specific placement preferences and incorporating these sets into the MILP formulation. STRL operators combine alternative placements and express combinatorial preferences, such as rack locality.
  - TetriSched integrates with the Rayon reservation system in Hadoop YARN, using reservation information, including estimated runtimes, deadlines, and reservation acceptance, to construct STRL expressions and prioritize jobs.
- Metrics / Baselines:
  - The authors measure deadline attainment for accepted SLO jobs, all SLO jobs, and SLO jobs without reservations, alongside mean BE completion latency. They also measure solver latency and total scheduling-cycle latency.
  - The main baseline is Rayon with YARN CapacityScheduler (Rayon/CS), with container preemption enabled to enforce reservation capacity guarantees. Both Rayon/CS and Rayon/TetriSched use Rayon for reservation admission.
  - On a real 256-node cluster with a production-derived SLO + BE workload, TetriSched provides higher SLO attainment and lower mean BE latency than Rayon/CS across the tested estimation errors. It meets over 95% of accepted SLO deadlines even when estimated runtimes are half their actual values (Figure 6).
  - The authors compare full TetriSched with TetriSched-NH (no heterogeneity awareness), TetriSched-NG (greedy scheduling instead of global scheduling), and TetriSched-NP (no plan-ahead) to evaluate each feature’s contribution.
  - On a real 80-node cluster with a synthetic homogeneous SLO + BE workload, TetriSched generally improves SLO attainment and BE latency. However, at −50% estimation error, its mean BE latency is approximately 3× higher than Rayon/CS because underestimated BE jobs occupy resources longer without preemption (Figure 8).
  - Figure 12 examines solver and scheduling-cycle latency as the plan-ahead horizon increases. Larger-scale results use simulations, including 1,000-node and 10,000-node clusters.
- Key insight and enabling idea: Greedy scheduling can make locally beneficial allocations that reduce overall SLO attainment. Delaying some jobs or assigning them less-preferred resources may produce a better global schedule. Estimated runtimes and placement preferences allow TetriSched to compare these alternatives through plan-ahead, while regular replanning adapts provisional allocations to new arrivals and runtime estimation errors.
- Novelty and impact: TetriSched combines global scheduling, runtime-aware plan-ahead, and regular replanning to exploit resource-type and start-time flexibility. STRL provides a general way to express space-time placement preferences and translate them into a MILP formulation, extending beyond schedulers with fixed preference structures. Its integration with Rayon in Hadoop YARN demonstrates how reservation information can improve runtime scheduling, with real-cluster experiments showing higher SLO attainment and lower best-effort latency under many tested conditions.
- Technical Qualities:
  - Framing and assumptions: The paper studies heterogeneous clusters serving a mix of SLO and BE jobs. It assumes that applications or production tools can provide estimated job runtimes for different resource types and quantities, along with deadlines and placement preferences. These estimates allow the scheduler to anticipate future resource availability, although they may be inaccurate.
  - Merits of the technical contributions: Global scheduling and plan-ahead allow TetriSched to compare immediate allocations with delayed or less-preferred placements, improving resource utilization and SLO attainment. Regular replanning adapts pending-job schedules to new arrivals and runtime estimation errors, while STRL provides a general way to express placement preferences.
  - Does the evaluation support claims and reveal limitations of the proposed approach?: Real-cluster experiments with production-derived and synthetic workloads support the scheduling benefits under the tested conditions. However, comparing Rayon/CS with preemption both enabled and disabled would better clarify the tradeoff between reservation enforcement and BE latency. Figure 8 reveals that severe runtime underestimation can worsen BE latency because TetriSched does not preempt running jobs, while Figure 7 uses a narrower estimation-error range without explaining the choice. The evaluation varies runtime estimates rather than testing an actual predictor and does not establish performance across many GPU models and CPU configurations. Sensitivity to the fixed utility weights and BE tail latency are also insufficiently characterized. Figure 12 reports scheduling latencies exceeding the four-second cycle period without clearly explaining overrun handling, and larger-scale results rely on simulation.
- Criticism:
  - Comparing Rayon/CS with preemption both enabled and disabled would better clarify the tradeoff between reservation enforcement and BE latency.
  - Figure 8 reveals that severe runtime underestimation can worsen BE latency because TetriSched does not preempt running jobs. At −50% estimation error, its mean BE latency is approximately 3× higher than Rayon/CS in the synthetic homogeneous workload.
  - The evaluation varies runtime estimates rather than testing an actual predictor, so the accuracy and error patterns of estimates in deployment remain uncertain.
  - Larger-scale results rely on simulation, so comparable scheduling quality and overhead on physical 1,000-node or 10,000-node clusters are not established.
  - Sensitivity to the fixed utility weights—1,000× the initial BE value for accepted SLO jobs and 25× for unreserved SLO jobs—is insufficiently characterized.
  - The evaluation reports mean BE completion latency but does not report tail latency or its distribution, leaving the delays experienced by the slowest BE jobs unclear.
  - Figure 12 reports scheduling latencies exceeding the four-second cycle period without clearly explaining how overruns affect subsequent scheduling cycles.
  - Figure 12 shows substantial overhead outside the solver for TetriSched-NG, but does not provide a detailed breakdown explaining that overhead.
  - Figure 7 tests a narrower estimation-error range (−20% to +20%) than Figures 6 and 8 (−50% to +100%) without explaining the choice, limiting evidence of robustness under severe errors for SLO-only workloads.
  - The evaluation does not establish performance across many GPU models and CPU configurations. Greater hardware diversity may increase placement alternatives, resource partitions, and MILP complexity.
  - STRL cannot express dependencies between pending jobs or end-to-end deadlines for job pipelines, limiting its applicability to multi-stage workflows.













