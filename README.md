# SMC_ADAE
Artifact description for the SMC paper.

### This repository contains the following files
* $A_1$: Input and output files for Exp.\;1: 3D parallelism in SMC-GPU.
* $A_2$: Input and output files for Exp.\;3: SMC-NPU.
* $A_3$: Input and output fils for Exp.\;2: Strong and weak scaling of SMC-GPU.
* $A_4$: Atom position data for the simulated APT needle specimen.

### A_1
Input:
```bash
   sbatch -N 2 --gres=gpu:8 --qos=gpugpu ./run_para.sh
```
`run_para.sh`:
```bash
   #!/bin/bash
   module purge
   module load openmpi/4.1.5_ucx1.17.0_nvhpc24.9_cuda12.4
   #make clean
   make
   export NCCL_DEBUG=INFO
   export NCCL_IB_DISABLE=0
   export NCCL_IB_HCA=mlx5_0:1,mlx5_1:1,mlx5_3:1,mlx5_4:1
   export NCCL_SOCKET_IFNAME=bond0
   export NCCL_IB_TIMEOUT=23
   export NCCL_IB_RETRY_CNT=7
   export OMPI_MCA_btl_openib_allow_ib=true
   export OMPI_MCA_btl=openib,self,vader  
   nvidia-smi | grep NVIDIA
   (
    for i in {1..1000}
    do
      date
      nvidia-smi | grep MiB
    done
   ) &
   mpirun -np 16 -N 8 ./ising_basic_3d -x 84 -y 630 -z 630 -n 100 -m 200 -a 2000 -i 950 -d 100 -o 1
```

Output: See [output files A1](./log_1B_EPI_billionSteps_GPU/) for details.

### A_2
Input:
```bash
   mpirun -n 8 -x UCX_LOG_LEVEL=error ./ising_basic_3d -x 500 -y 220 -z 220 -n 10000 -m 10 -a 2000 -i 850 -d 50 -o 1
```
Output: See [output files A2](./log_1B_EPI_billionSteps_GPU/) for details.

### A_4
1. Monte Carlo simulation of $\rm{Fe_{29}Co_{29}Ni_{28}Al_{7}Ti_{7}}$ using a one-billion-atom supercell. The results are obtained with Exp. 1. The supercell size is:  The following GIF file only shows the XY face due to the complete 1 billion atoms is too larger to show. 
![Configuration vs MC steps](./lattice0.gif)
