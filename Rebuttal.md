## P1: The SMC-NPU Algorithm
Note that SMC-GPU are designed for the SIMT programming model in GPU. The vector unit in Ascend NPU adopts the SIMD programming model.
#### Motivation 
The NPUs are specifically designed for deep learning tasks. Compared to SIMD vectorized instruction sets on CPUs, the SIMD computation of the Ascend chip's vector processing unit has another notable characteristic: data reads and writes must be memory-contiguous or segment-contiguous, which poses significant programming challenges for general HPC tasks. Note that both CPUs and GPUs can gather and assemble non-contiguous scalar data, whereas the Ascend chip cannot.

![alt text](image.png)

Specifically, Monte Carlo simulations generally exhibit random memory access patterns. For simplicity, we use a 2D square lattice for discussion, as shown in Fig.1. In an atom-swap trial in SMC, while the red atoms follow a regular pattern, their exchange neighbors (orange atoms) are randomly distributed in space. In CPU and GPU architectures, each red atom attempts to swap randomly with its nearest neighbor. Each CPU core or GPU thread executes identical exchange instructions while reading atom-specific data from different memory locations - an operation that poses little challenges for conventional architectures, but presents serious computational bottleneck for NPUs due to the abscence of real memory cache system.

To address the above problem, we propose the SMC-NPU algorithm, which is designed to unleash the computing power in NPU for MCMC atomistic simulation by **coalescing memory access pattern, as much as possible, and vectorize the scalar operations via masked operations**. We will explain the two optimization techniques in detail in the following.

#### Memory Access Coalescing
In SMC-GPU, only one lattice are used, and all swap trials are attempted on-site. While such a practice saves memory space, it inevitablly introduces frequent read and write operations that demand irregular memory access, which is highly unfavorable for NPU.
![alt text](image-1.png)
To coalescing the memory access, the SMC-NPU algorithm uses **two lattices** to record the atomic configuratoins before and after the swap trials, respectively, as shown in Fig. 6 of the manuscript. Note that the usage of of two lattices enables the **decoupling of the two important steps: energy calculation, and Metropolis updating**, as illustrated in Fig. 5. As a result, the most computing-intensive step, the calculation of the local energies for each sites, can be done in an **embarassively-parallel way, with contigious memory access**. While some overhead still exists due to the need to padding the neighbor vectors for the hardware's SIMD width, the contigious memory access pattern, as well as the huge parallel degrees introduced by SMC-NPU, can effectively utilize the large number (20) of 2048-bit SIMD vector units in an NPU. Note that the lattice are stored as Int16, therefere a total of 128*20=2560 sites can be processed simultaneously in a clock cycle by a single  **TBD with Kai**. 
![alt text](image-2.png)
Note that unlike the masked operations to be discussed below, the above optimization technique in principle can be applied to any vector-based accelerators other than NPUs.
#### Vectorization via Masked Operations

