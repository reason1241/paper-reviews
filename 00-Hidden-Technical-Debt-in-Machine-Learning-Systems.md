# Hidden Technical Debt in Machine Learning Systems

Date: 2026-08-28

- [Link](https://proceedings.neurips.cc/paper_files/paper/2015/file/86df7dcfd896fcaf2674f757a2463eba-Paper.pdf)

Like software engineering, machine learning (ML) systems can accumulate a lot of technical debts. This paper explains these issues and considers possible mitigations.

- One of the input features' distribution can change or new features can be added. Not only inputs, but also other configurations can affect the model, requiring us to change existing ML models. For this, 1) decomposition the problem into sub-problems and applying ensemble technique and 2) change detection may help. However, in the first case, improving one component may make the overall system worse if its errors become more correlated with those of other components.
- If we use a base model for similar model and this happens lin a chain (cascade), it may cost a lot in the end. We may make a separate model to reduce dependencies.
- Model outputs can be silently utilized by other systems, so changing a model can unexpectedly affect those systems.
- Input data can change due to ownership changes, data improvement, etc., which can negatively affect the existing model's results. This can be controlled using versioned copies.
- Redundant, grouped, extra feature, and correlated features have negative effects.
- Like compilers analyzing code dependencies, static analysis can benefit data dependencies.
- An ML model can affect itself (direct feedback loop), or different models can affect each other (hidden feedback loop), so chaning one of them can affect future behavior or other systems.
- Adding glue code to use general purpose solution can make the system complicated.
- Data pipelines can become unnecessarily complex as new data sources and processing steps are added.
- Code for expermentation can remain in the system even if it is no longer used.


- There is no strong abstraction for ML systems, making them more complicated.
- Plain raw data types, the combination of multiple languages, and prototypes integrated into production systems can make the system difficult to maintain.
- Too many configurations are problematic. It would be good to reduce unnecessary configurations and make configurations easy to verify.
- The world ever changes, so fixed thresholds may not work in the future. We may need to re-evaluate the model using hold-out data.
- Checking input-output distributions, action limits (the number of prediction or ratio), and monitoring upstream process can be beneficial.
- Basic sanity checks, reproducability, process management (using visualization tools etc.), and a culture that values reducing unnecessary complexity can be crucial.
