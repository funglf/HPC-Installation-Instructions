# HPC Package Installation Guide 

## Purpose
This guide is for installation of commonly used packages in HPC/numerical computing. It is applicable for both HPC environments and PC given they are linux based.

## Installation Practice in this guide
### Directories
It is recommended that the source code/ source packages be downloaded in a separate directory to the installation directory to keep things clear. The guide will also provide writing of a modulefile in Lmod fashion -- the newer modulefile system.
### Naming Convention
As I prefer to keep packages and dependencies clear and explicit, naming of a package could be long.
For instance Scalapack Version 2.2.2, built with gcc 13.4.0, openmpi 5.0.8 and openblas 0.3.29
```bash
scalapack/2.2.2-gcc13.4.0-openmpi5.0.8-openblas0.3.29
```
This communicates all the different versions of dependencies it uses.
Feel free to change it to a simpler version for your installation.

## Problems
Feel free to raise an issue if there are issues with the guide. The guide is not intended to be a "one-size-fits-all". There could be nuances to each individual systems or hardware that could have potential issues. 