# HPC Package Installation Guide 

## Purpose
This guide provides step by step instructions for manually installing commonly used software packages for High Performance Computing (HPC) and Scientific Computing. Every guide has been verified on several production HPC clusters -- it should also work in Linux Personal Workstations. Official documentation of packages are often intentially generic, this guide provides the additional specific configuration details required.

## Why Manual Installation
-   Tuning of software packages to the specific hardware environment. 
-   Dependency testing/benchmarking -- different dependencies may have significant impact on performance, eg dependency on BLAS libs.
-   Shorter Build times -- package manager like SPACK often takes long time to build as it is defaulted to build everything starting from compilers

## Installation Practice 
### Separation of Build and Install
It is recommended to separate the build tree and the final installation location/prefix. 

### Directory/Modulefile naming Convention
Package names embeds the version and important dependencies for clear communication especially in shared environments.

#### Example
For instance Scalapack Version 2.2.2, built with gcc 13.4.0, openmpi 5.0.8 and OpenBLAS 0.3.29, is installed into:
```bash
scalapack/2.2.2-gcc13.4.0-openmpi5.0.8-openblas0.3.29
```
While scalapack built with Intel MKL is:
```bash
scalapack/2.2.2-gcc13.4.0-openmpi5.0.8-mkl2024.0
```
This convention makes dependencies explicit and communicates without any extra documentation. Should a shorter naming convention is required, please edit the `BASENAME` portions of the guides.

## Issues and Feedback
If you encounter problems or any undocumented edge cases, please open an issue in the repository. Contributions and pull requests that extend or refine these instructions are welcomed!!