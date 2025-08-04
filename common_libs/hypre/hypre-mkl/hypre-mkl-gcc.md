# Hypre Installation Guide using MKL with GNU compilers


## Define Directories
```bash
BUILD_DIR= # Where is the source code
SW_DIR= # Where do you want to install
MODULE_DIR= # Where your modulefiles are
```

## Setup Environment
### Choose Compiler
>In HPC environment:
>```bash
>module load gcc # Or equivalent
>```

```bash
export CC=gcc
export CXX=g++
export FC=gfortran

export CC_VERSION=$($CC --version | head -n 1 |  awk '{ print $3}')
COMPILER=${CC}${CC_VERSION}
```


### Choose mkl Version 
```bash
MKL_VERSION=25.1 # Or any other valid versions
MKLROOT= # where you installed mkl -- defined when you module load Intel MKL
```


### Choose MPI
You could have different implementations and versions of MPI. Please use a name that is clear to you. For instance using OpenMPI v5.0.7:
```bash
MPI=ompi5.0.7
```
Or eg. mvapich v2.3.0:
```bash
MPI=mvapich2.3.0
```

### Choose `mpicc` wrapper Host Compiler
In OpenMPI:
```bash
export OMPI_CC=${CC}
export OMPI_CXX=${CXX}
export OMPI_FC=${FC}
```

> Note: Not every MPI has an environment variable to choose the compiler. For instance, Intel MPI requires a compiler flag when compiling `-{cc,cxx,fc}=<compiler>`. Please check your MPI implementation on how this is done.

### Choose Hypre Version
```bash
HYPRE_VERSION=2.33.0
```

### Define install location
```bash
HYPRE_BASENAME=${HYPRE_VERSION}-${COMPILER}-${MPI}-mkl${MKL_VERSION}
HYPRE_ROOT=${SW_DIR}/hypre/${HYPRE_BASENAME}
```

## Build
### Download source code
```bash
cd ${BUILD_DIR}
wget https://github.com/hypre-space/hypre/archive/refs/tags/v${HYPRE_VERSION}.tar.gz
tar -xvf v${HYPRE_VERSION}.tar.gz
```

## Configure 
```bash
cd ${BUILD_DIR}/hypre-${HYPRE_VERSION}/src

./configure --prefix=${HYPRE_ROOT} \
            --enable-shared \
            --with-MPI \
            --with-blas-lib-dirs="${MKLROOT}/lib" \
            --with-blas-libs="mkl_rt" \
            --with-lapack-lib-dirs="${MKLROOT}/lib" \
            --with-lapack-libs="mkl_rt" \
            --with-openmp 
```

## Build and Install
```bash
make -j 8
make install # copies the compiled binaries and libs into desired location
```

## Modulefile -- for HPC users

### Create Directory
```bash
mkdir -p ${MODULE_DIR}/hypre
```

### Write Modulefile
```bash
tee ${MODULE_DIR}/hypre/${HYPRE_BASENAME}.lua > /dev/null <<EOF
help([[
HYPRE ${HYPRE_VERSION}
===============

Built with:
- ${COMPILER}
- ${MPI}
- mkl ${MKL_VERSION}

Installed: $(date +"%d %B %Y")
]])

whatis("Name: Hypre")
whatis("Version: ${HYPRE_VERSION}")

local hypre_root = "${HYPRE_ROOT}"

prepend_path("LD_LIBRARY_PATH",     pathJoin(hypre_root, "lib"))
prepend_path("LIBRARY_PATH",        pathJoin(hypre_root, "lib"))
prepend_path("PKG_CONFIG_PATH",     pathJoin(hypre_root, "lib/pkgconfig"))
prepend_path("CPATH",               pathJoin(hypre_root, "include"))

setenv("HYPRE_ROOT", hypre_root)
EOF
```

>Note: It is recommended to add "load(mkl, yourmpi)" into the modulefile to automatically load the dependencies. Not added in example as you might have a differnet name or system to it.