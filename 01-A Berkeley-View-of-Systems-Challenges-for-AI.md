# A Berkeley View of Systems Challenges for AI

Date: 2026-08-29

- [Link](https://arxiv.org/pdf/1712.05855)

- This paper covers current and promising research directions based on developments in ML systems.
- They attribute the improvement of ML/AI to 1) big data, 2) big systems (e.g. Hadoop) and 3) accessability to technology (e.g. PyTorch, cloud computing resources)
- They suggest four challenges: 1) Mission-critical AI (their role and danger are critical), 2) personalized AI, 3) AI across organizations (organizations can't or don't want to share data for training etc.), and 4) AI demands outpacing hardware improvements.
- They suggest following research opportunities:
  - The world continuously changes, so AI models should adapt to it.
    - Since the world continuously changes, batch training may not be appropriate. Continual learning and online learning can be adopted, but traditional online learning does not fully cover cases where agents' actions change the environment. Reinforcement learning may be useful, but building large-scale systems for RL simulation can be challenging. Also, simulated reality (SR) may help agents by simulating possible outcomes before they act.
    - In mission-critical settings, decisions must be reliable to prevent disasters. For this, tracking data provenance may help by reducing uncertainty about data sources. Two challenges are robustness against 1) noisy and adversarial feedback 2) unforeseen and adversarial inputs. Handling cases that the model have never experienced can also be a good research topic (e.g. human intervention in autonomous driving).
    - Making AI decisions explainable is also crucial. For this, changing inputs/configurations with faithful replay can help determine the impact of changes.
  - Security is also important. Attackers may take over control of the system or extract confidential training data or models.
    - Secure execution environments such as  Intel’s Software Guard Extensions (SGX), or sandboxing may help. However, minimizing the amount of code running inside the trusted computing base is crucial because there are a lot of limitations (e.g. available functionality and cost).
    - Attackers may interfere with two processes: 1) data poisoning (during training) and 2) evasion attacks (by manipulating input data during inference). Data source tracking and input validation/detection may help.
    - Organizations may not want to share their confidential data even if sharing may benefit everyone. Secure multi-party computation (MPC, which enables multiple parties to jointly compute without revealing their private inputs) may be helpful, but it comes with substantial overhead. Differential privacy (adding noise to protect information from being inferred from outputs) can help. Lastly, privacy can be time dependent (e.g. historical stock data may be less sensitive, while current bidding data may be valuable). Providing incentives to participants may give reasons to participate.
  - Specialized syetems and hardwares may help (AI-specific architecture)
    - Google's Tensor Processing Units (TPUs) or 3D XPoint from Intel and Micron can be examples. New software architectures may also help systems efficiently use heterogeneous, domain-specific hardware.
    - Like software engineering, composable models and modularity may help.
    - Given the expected large discrepency between the capabilities of datacenters and edge devices due to the differences in size and power constraints, a mix of cloud and edge devices would be promising (e.g. running large models in datacenter and small models on cellphones). New software technology for heterogeneous devices or advanced compliers and just in time (JIT) compilation on edge devices would be helpful. Lastly, since we have too much data, data compression technology may help.
