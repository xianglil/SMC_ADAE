# SMC_ADAE
Artifact description for the SMC paper.

### This repository contains the following files
* $A_1$: Input and output files for Exp.\;1: 3D parallelism in SMC-GPU.
* $A_2$: Input and output files for Exp.\;3: SMC-NPU.
* $A_3$: Input and output fils for Exp.\;2: Strong and weak scaling of SMC-GPU.
* $A_4$: Atom position data for the simulated APT needle specimen.

### A_4
1. Monte Carlo simulation of $\rm{Fe_{29}Co_{29}Ni_{28}Al_{7}Ti_{7}}$ using a one-billion-atom supercell. The results are obtained with Exp. 1. The following GIF file only shows the XY face due to the complete 1 billion atoms is too larger to show. 
![Configuration vs MC steps](./lattice0.gif)
2. The original data for the APT specimen, as well as the selected nanoparticles shown in the paper, can be accessed with this [Google Drive Link](https://drive.google.com/file/d/1L-rYCH0CVspZcDoBzcBFmOcWfooLe6Ob/view?usp=drive_link).

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
Output: See [output files A2](./log_table4_910B/) for details.
    
### A_3
Input and Output: See the results at [Strong Scaling](log_strong_scaling.zip) and [Weak Scaling](log_weak_scaling.zip)

#### Strong scaling:
```bash
   sbatch --gpus=1 ./run_para.sh
   sbatch --gpus=2 ./run_para.sh
   sbatch --gpus=4 ./run_para.sh
   sbatch --gpus=8 ./run_para.sh
   sbatch -N 2 --gres=gpu:8 --qos=gpugpu ./run_para.sh
   sbatch -N 4 --gres=gpu:8 --qos=gpugpu ./run_para.sh
```
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
   mpirun -n 1 ./ising_basic_3d -x 960 -y 510 -z 525 -n 100 -m 1 -a 2000 -i 2000 -d 50 -o 1
   #mpirun -n 2 ./ising_basic_3d -x 480 -y 510 -z 525 -n 100 -m 1 -a 2000 -i 2000 -d 50 -o 1
   #mpirun -n 4 ./ising_basic_3d -x 240 -y 510 -z 525 -n 100 -m 1 -a 2000 -i 2000 -d 50 -o 1
   #mpirun -n 8 ./ising_basic_3d -x 120 -y 510 -z 525 -n 100 -m 1 -a 2000 -i 2000 -d 50 -o 1
   #mpirun -np 16 -N 8  ./ising_basic_3d -x 60 -y 510 -z 525 -n 100 -m 1 -a 2000 -i 2000 -d 50 -o 1
   #mpirun -np 32 -N 8  ./ising_basic_3d -x 30 -y 510 -z 525 -n 100 -m 1 -a 2000 -i 2000 -d 50 -o 1
```

| Model | LinkCell | Total Atoms | 1 GPU time(efficiency) | 2 GPU time(efficiency) | 4 GPU time(efficiency) | 8 GPU time(efficiency) | 16 GPU time(efficiency) |
| :----: | :----: | :----: | :----: | :----: | :----: | :----: | :----: |
| EPI | 3x3x3 | 1B | 0.283676s(1.0) | 0.152148s(0.932) | 0.0835246s(0.849) | 0.0497673s(0.713) | 0.0522457s(0.339) |
| EPI | 5x5x5 | 1B | 0.31834s(1.0) | 0.211838s(0.751) | 0.147654s(0.539) | 0.116952s(0.340) | 0.296601s(0.0671) |
| ML | 4x4x4 | 1B | 27.1809s(1.0) | 13.7398s(0.989) | 6.80817s(0.998) | 3.47778s(0.977) | 1.81881s(0.934) |
| ML | 6x6x6 | 1B | 27.2531s(1.0) | 13.8396s(0.985) | 6.90118s(0.987) | 3.57882s(0.952) | 2.13979s(0.796) |

* 1~16 H800 GPUs
* Model: EPI && MLIP
* Total number of atoms: ~ 1B


#### Exp. 3: weak scaling
* Input:
```bash
sbatch --gpus=1 ./run_para.sh
sbatch --gpus=2 ./run_para.sh
sbatch --gpus=4 ./run_para.sh
sbatch --gpus=8 ./run_para.sh
sbatch -N 2 --gres=gpu:8 --qos=gpugpu ./run_para.sh
sbatch -N 4 --gres=gpu:8 --qos=gpugpu ./run_para.sh
```

```run_para.sh
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

mpirun -n 1 ./ising_basic_3d -x 960 -y 510 -z 525 -n 100 -m 1 -a 2000 -i 2000 -d 50 -o 1
#mpirun -n 2 ./ising_basic_3d -x 960 -y 510 -z 525 -n 100 -m 1 -a 2000 -i 2000 -d 50 -o 1
#mpirun -n 4 ./ising_basic_3d -x 960 -y 510 -z 525 -n 100 -m 1 -a 2000 -i 2000 -d 50 -o 1
#mpirun -n 8 ./ising_basic_3d -x 960 -y 510 -z 525 -n 100 -m 1 -a 2000 -i 2000 -d 50 -o 1
#mpirun -np 16 -N 8  ./ising_basic_3d -x 960 -y 510 -z 525 -n 100 -m 1 -a 2000 -i 2000 -d 50 -o 1
#mpirun -np 32 -N 8  ./ising_basic_3d -x 960 -y 510 -z 525 -n 100 -m 1 -a 2000 -i 2000 -d 50 -o 1
```
* [Output files](./log_weak_scaling/)

| Model | LinkCell | Atoms Per GPU | 1 GPU time(efficiency) | 2 GPU time(efficiency) | 4 GPU time(efficiency) | 8 GPU time(efficiency) | 16 GPU time(efficiency) |
| :----: | :----: | :----: | :----: | :----: | :----: | :----: | :----: |
| EPI | 3x3x3 | 1B | 0.283676s(1.0) | 0.291178s(0.974) | 0.292705s(0.969) | 0.293911s(0.965) | 0.310069s(0.915) |
| EPI | 5x5x5 | 1B | 0.31834s(1.0) | 0.353998s(0.899) | 0.360995s(0.882) | 0.36677s(0.868) | 0.519045s(0.613) |
| ML | 4x4x4 | 1B | 27.1809s(1.0) | 27.216s(0.999) | 27.2803s(0.996) | 27.576s(0.986) | 27.6062s(0.985) |
| ML | 6x6x6 | 1B | 27.2531s(1.0) | 27.6067s(0.987) | 27.0458s(1.01) | 27.7111s(0.983) | 27.9004s(0.977) |

* 1~16 H800 GPUs
* Model: EPI && MLIP
* Number of atoms per GPU: ~ 1B

#### Throughput estimation
- EPI table1 `log_weak_scaling/FCC_EPI_dist/out_fcc_epi_dist_333_16/out0.dat`
  - TP `16*960*510*525*4/(31.0069/100)=5.31*10^10`
  - TP per chip `960*510*525*4/(31.0069/100)=3.31*10^9`

- ML table2  `log_weak_scaling/FCC_MLIP_dist/out_fcc_mlip_dist_444_16/out0.dat`
  - TP `16*1152*468*468*4/(276.062/10)=5.85*10^8`
  - TP per chip `1152*468*468*4/(276.062/10)=3.66*10^7`
- ML table2  `log_table2`
  - TP `16*1152*468*468*4/(27065.2/1000)=5.97*10^8`
  - TP per chip `1152*468*468*4/(27065.2/1000)=3.72*10^7`

- ML table3 A800  `log_table3`
  - TP `1152*468*468*4/(405.802/10)=2.49*10^7`
 
- ML table4 NPU 910B `log_table4_910B`
  - TP `8*1152*468*468*4/(1302.93/10)=6.20*10^7`
  - TP per chip `1152*468*468*4/(1302.93/10)=7.75*10^6`

