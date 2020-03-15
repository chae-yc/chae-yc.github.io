---
title: "Parallel process management - MPI"
categories:
  - scientist
  - blogging
tags:
  - MPI
  - HPC
last_modified_at: 2020-02-12T23:58:00+09:00
toc: true
published: true
share: false
---

## Termiology

SMS: System management stack

MPI: Message Passing Interface
PMI: Process Management Interface <- part of MPICH(in detail PMI-1 is implemented in MPD)
PMIx: PMI Exascale

RTE: Run-Time Environment

Hybrid (e.g. MPI+OpenMP) programming

## HPC ecosystem

![Arm-HPC-Ecosystem-overview](/assets/images/2020-02-12-Parallel_process_management/Arm-HPC-Ecosystem-overview.png)

[image source](https://community.arm.com/developer/tools-software/hpc/b/hpc-blog/posts/arm-entering-the-high-performance-computing-market)

## MPI standard

### Classification

![classification](/assets/images/2020-02-12-Parallel_process_management/classification.png)

[imagesource](https://www.slideshare.net/rcastain/exascale-process-management-interface) p.3
- Parallel programming library
  - OMPI (part of OpenMPI)
  - MVAPICH2 (based on MPI 3.1)
  - Intel MPI
  - SiCortex MPI
  - Microsoft MPI
- Process Managers (using PMI library)
  - Hydra, MPD, SMPD, Gforker, Remhsell (part of MPICH, MPICH2) -- PMI-1, PMI-2 is developed for this project
  - ~~ORTEd~~ PPRTE(PMIx Reference RunTime Environment) (part of OpenMPI)
  - Slurm PMI (part of Slrum)
  - OpenPMIx
- Process manager
  - Slurm
  - IBM LSF
  - TORQUE
  - Unix (ssh/rsh)

## Container Comparison

![comparison](/assets/images/2020-02-12-Parallel_process_management/comparison.png)
Kurtzer, Gregory & Sochat, Vanessa & Bauer, Michael. (2017). Singularity: Scientific containers for mobility of compute. PLoS ONE. 12. 10.1371/journal.pone.0177459. 

Singularity **extreme portability** 