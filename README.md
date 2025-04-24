# SMC_ADAE
Artifact description and artifact evaluation for SMC
### This repository contains the following files
1. A gif movie of the configurations changes with the Monte Carlo simulation steps and temperatures.
2. The original input and output file for the results of the paper.
3. The calculation steps for generating the tables.

### Movie
Monte Carlo simulation of $\rm{Fe_{29}Co_{29}Ni_{28}Al_{7}Ti_{7}}$ using a one-billion-atom supercell. The results are obtained with Exp. 1. The supercell size is:  The following GIF file only shows the XY face due to the complete 1 billion atoms is too larger to show. 
![Configuration vs MC steps](./lattice0.gif)

### Input and output
A table ?
#### Exp. 1: 1-billion atoms, EPI, up to $3\times 10^6$ MC steps
* Input:
* [Output files](./log_1B_EPI_billionSteps_GPU/)
* A total of 16 H800 GPUs
* Lattice parallelization degree: 8
* Replica-exchange degree: 2
* Model: EPI
* Total number of atoms: $8 \times 4\times 84 \times 630 \times 630 = 1,066,867,200$
* MC steps and schedule: Temperature decrease from 2000K to 10000 K, with a T-step of 100 K. See the [output files](./log_1B_EPI_billionSteps_GPU/) for details.
