# PyTorch: An Imperative Style, High-Performance Deep Learning Library

Date: 2026-08-30

- [Link](https://arxiv.org/abs/1912.01703)

- Problem: Existing deep learning frameworks often trade off usability and speed. Static dataflow graph-based frameworks provides visibility into the whole computation ahead of time and performance & scalability improvement, while those are 1) hard to use, 2) difficulty of debugging, and 3) inflexibility. Existing define-by-run approach shows less performance or expressiveness.
- Solution: 
  - The authors propose a machine learning library PyTorch, which satisfies both.
  - They focused on 1) following Python language principles, 2) hiding complexity behind intuitive APIs, and 3) balancing performance and simplicity instead of focusing only on simplicity.
  - Unlike other graph based approaches, PyTorch adopts an imperative, define-by-run approach, which is familiar to Python users.
  - It is highly interoperable with the Python ecosystem. For example, PyTorch tensors and NumPy arrays can exchange data without copying the underlying memory.
  - They implemented efficient automatic differentiation using operator overloading and reverse-mode automatic differentiation. For safe tensor mutation, they use 1) a tensor versioning system and 2) avoid copy-on-write.
- Metrics / Baselines: 
  - They used training speed (throughput) with 6 models (AlexNet, VGG-19, ResNet-50, MobileNet, GNMTv2, and NCF) on 6 frameworks (Chainer, CNTK, MXNET, PaddlePaddle, TensorFlow, and PyTorch). PyTorch showed the fastest speed on 3 out of 6 models and was within 17% of the fastest framework on all benchmarks.
  - They also counted the number of mentions of PyTorch in papers on arXiv. The percentage of mentioning PyTorch among machine learning tools grew up to approximately 50% on arXiv in two years.
  - The authors evaluate GPU execution behavior using profiler traces on ResNet-50, showing that CPU-side scheduling can run ahead of GPU execution and achieve high GPU utilization.
- Key insight and enabling idea: Usability and interoperability are important. Python users are familiar with Python programs, classes, control flow, and debugging tools, so using Pythonic imperative execution while offloading expensive computation to optimized C++/CUDA libraries can provide both usability and high performance.
- Claimed technical contributions: They implemented 1) C++ core utilizing YAML meta-data files binding, 2) control and dataflow separation by leveraging CUDA stream, 3) a custom caching tensor allocator using CUDA APIs and one-pool-per-stream, 4) a multiprocessing module using shared memory, and 5) reference counting.
- Novelty and impact: Authors implemented Python-friendly machine learning framework with competitive performance, and almost 50% of papers mentioned PyTorch.
- Technical Qualities:
  - Framing and assumptions: ML practitioners use Python a lot due to its openness and large ecosystem, so high interoperability with the existing system must be important.
  - Merits of the technical contributions: High usability with comparable performance.
  - Does the evaluation support claims and reveal limitations of the proposed approach?: If the arXiv mention metric is a reasonable proxy for real usage, the rapid adoption supports their assumption that usability and Python integration are important. Also, the presented workloads show that PyTorch can achieve performance comparable to other frameworks.
- Weakness: The evaluation mainly focuses on single-machine, single-GPU workloads and does not provide much evidence about distributed scalability. The authors also mention improving distributed computation as future work.
