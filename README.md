# Tensor Program Hyperparameter Search Parallelization with Shared Address Space and Message Passing Models

## Links
[Project Proposal (PDF)](./proposal.pdf)

[Project Milestone (PDF)](./milestone.pdf)

[Final Project Report (PDF)](./FinalProjectReport.pdf)

[Poster (PDF)](./Poster.pdf)

## Team
Joshua Hong (jjhong), Kit Ao (kitao)

## Summary

We are going to parallelize the DFS-based hyperparameter search algorithm of the Mirage tensor optimizer (Wu et al) using a Shared Address Space model (via OpenMP) and a Message Passing model (via MPI). We are going to compare these two methods (in addition to static assignment via pthread-based multithreading) in terms of workload balancing, resource contention, as well as overall runtime improvements.

## Background

Effective search for GPU kernel parameters enables the generation of fast GPU kernels that significantly improve the performance of tensor programs. Mirage is a tensor optimizer that performs an exhaustive search over the kernel, thread block, and thread levels in order to find configurations (known as 𝜇Graphs) that are equivalent to the input tensor program. The search process can be viewed as a DFS. Pruning is performed by invoking a probabilistic equivalence verification process that verifies whether a given 𝜇Graphs is equivalent to or a subset of the input program. Overall, Mirage divides the input tensor program into LAX subprograms, consisting of multi-linear operations (matrix multiplication, convolution), division, as well as exponentiation. It then performs the search on each of these LAX subprograms. The entire optimization process is bounded by the search process. Currently, a multithreaded approach based on interleaved static workload assignment is used as the parallelization strategy. However, as we will show, this current approach suffers from workload imbalance as well as resource contention.

## The Challenge
There are several challenges to implementing parallelism on the Mirage search process.
1. The DFS search algorithm is not an intuitive algorithm to parallelize. This is because in a DFS, consecutive nodes are explored sequentially. This is unlike a BFS where the nodes of the same level of the search are independent of each other and can be processed in parallel.
2. During the search process, if a node’s configuration fails a verification step, the node gets pruned. Due to this, the search times of nodes at the same level can differ significantly since some nodes could be pruned very early on. This means that a static assignment of work based on partitioning the higher-level nodes would result in workload imbalance.
3. The search time is bound by a compute-intensive verification step that invokes the z3 solver. The black-box nature of the z3 solver makes it very difficult to optimize for the Mirage search process. Additionally, contention on the z3 solver might explain the slowdown in speedup seen at higher thread counts.

## Resources

We plan on developing our Shared Address Space and Message Passing approaches on locally machines as well as the GHC machines that have GPUs. Additionally, we hope to have access to machines with more resources, such as the PSC machines. The code base we are starting from is the public Mirage repository, available [here](https://github.com/mirage-project/mirage) . Additionally, we reference the documentation page [here](https://mirage-project.readthedocs.io/en/latest/transpiler.html)  for more information as well as the original paper [here](https://arxiv.org/pdf/2405.05751). 

As mentioned before, we will utilize the OpenMP framework for the Shared Address Space implementation, and the MPI framework for the Message Passing implementation.

## Goals and Deliverables

### Plan to Achieve
- [x] Implement the Shared Address Space (OpenMP) and Message Passing (MPI) versions of the DFS search algorithm.
- [x] Benchmark the performance of the two versions in terms of speedup performance and workload distribution.
- [x] **Poster Session**: we will show graphs that demonstrate that our approaches provide significant speedup when compared to the existing implementation. 

### Hope to Achieve
- [ ] Investigate how to reduce contention on the z3 solver. Possibly ideas are caching to reduce unnecessary calls to the z3 solver, reducing calls to the z3 solver by limiting the number of verifications.
- [ ] Implement a BFS-based search process to see if that maps better to a parallel implementation.

## Platform Choice

As mentioned previously, we plan on developing our Shared Address Space and Message Passing approaches on locally machines as well as the GHC machines that have GPUs. We choose to use these machines for development as a GPU is required to use Mirage. We also hope to benchmark with higher thread counts to fully capture the scaling properties of our approaches, which may require the use of machines with more allocated resources. Our development will be done in C++, as the original codebase we are adding on to and the parallelization interfaces we plan on using are both in C++

## Schedule

| Week | Goals | Status/Assignment
|--------------------|------|------
|November 18 - November 24          | Shared Address Space approach | Completed
|November 25 - December 1          | Message Passing approach | Completed
|November 2 - December 5          | Complete Shared Address Space benchmarking | Kit
|November 6 - December 8          | Message Passing approach compatibility with PSC | Joshua
|November 9 - December 12          | Complete Message Passing benchmarking | Joshua
|December 13 - December 15         | Final Report | Kit and Joshua

## Citations

Wu, M., Cheng, X., Padon, O., & Jia, Z. (2024). A Multi-Level Superoptimizer for Tensor Programs. arXiv preprint arXiv:2405.05751.
