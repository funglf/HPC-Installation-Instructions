# OpenBLAS installation guide

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

### Choose OpenBLAS Version 
```bash
OPENBLAS_VERSION=0.3.29 # Or any other valid versions
```

### Define Installation Location
```bash
OPENBLAS_ROOT=${SW_DIR}/openblas/${OPENBLAS_VERSION}-${COMPILER}
```

## Build
### Download source code
```bash
cd ${BUILD_DIR}
wget https://github.com/OpenMathLib/OpenBLAS/archive/refs/tags/v${OPENBLAS_VERSION}.tar.gz
tar -xvf v${OPENBLAS_VERSION}.tar.gz
```

### CMake
```bash
cd ${BUILD_DIR}/OpenBLAS-${OPENBLAS_VERSION}

mkdir -p build
cd build       

cmake -DCMAKE_INSTALL_PREFIX=${OPENBLAS_ROOT} \
        -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_C_COMPILER=${CC} \
        -DCMAKE_CXX_COMPILER=${CXX} \
        -DCMAKE_Fortran_COMPILER=${FC} \
        -DBUILD_SHARED_LIBS=1 \
        -DUSE_OPENMP=ON \
        -DUSE_LAPACK=1 \
        ..
```

### Build & Install
```bash
make -j 8
make install # copies the compiled binaries and libs into desired location
```

## Modulefile -- for HPC users

### Create Directory
```bash
mkdir -p ${MODULE_DIR}/openblas
```

### Write Modulefile
```bash
tee ${MODULE_DIR}/openblas/${OPENBLAS_VERSION}-${COMPILER}.lua > /dev/null <<EOF
-- Module file for OpenBLAS ${OPENBLAS_VERSION} built with ${COMPILER}

help([[
OpenBLAS ${OPENBLAS_VERSION}
===============

Built with ${COMPILER}
Installed: $(date +"%d %B %Y")
]])

whatis("Name: OpenBLAS")
whatis("Version: ${OPENBLAS_VERSION}")

local openblas_root="${OPENBLAS_ROOT}"

prepend_path("LD_LIBRARY_PATH",     pathJoin(openblas_root, "lib"))
prepend_path("LIBRARY_PATH",        pathJoin(openblas_root, "lib"))
prepend_path("CPATH",               pathJoin(openblas_root, "include"))
prepend_path("PKG_CONFIG_PATH",     pathJoin(openblas_root, "lib/pkgconfig"))

setenv("OPENBLAS_ROOT", openblas_root)
EOF
```