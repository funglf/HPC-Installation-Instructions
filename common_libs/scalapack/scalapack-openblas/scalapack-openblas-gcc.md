# Scalapack Installation Guide using OpenBlas

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
OPENBLAS_ROOT= # where you installed openblas
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

### Choose Scalapack Version
```bash
SCALAPACK_VERSION=2.2.2
```

### Define install location
```bash
SCALAPACK_BASENAME=${SCALAPACK_VERSION}-${COMPILER}-${MPI}-openblas${OPENBLAS_VERSION}
SCALAPACK_ROOT=${SW_DIR}/scalapack/${SCALAPACK_BASENAME}
```

## Build
### Download source code
```bash
cd ${BUILD_DIR}
wget https://github.com/Reference-ScaLAPACK/scalapack/archive/v${SCALAPACK_VERSION}.tar.gz
tar -xvf v${SCALAPACK_VERSION}.tar.gz
```

### CMake
```bash
cd ${BUILD_DIR}/scalapack-${SCALAPACK_VERSION}

mkdir -p build
cd build       


cmake -DCMAKE_INSTALL_PREFIX=${SCALAPACK_ROOT} \
      -DCMAKE_C_COMPILER=${CC} \
      -DCMAKE_CXX_COMPILER=${CXX} \
      -DCMAKE_Fortran_COMPILER=${FC} \
      -DBUILD_SHARED_LIBS=ON \
      -DCMAKE_BUILD_TYPE=Release \
      -DLAPACK_LIBRARIES="${OPENBLAS_ROOT}/lib64/libopenblas.so" \
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
mkdir -p ${MODULE_DIR}/scalapack
```

### Write Modulefile
```bash
tee ${MODULE_DIR}/scalapack/${SCALAPACK_BASENAME}.lua > /dev/null <<EOF
help([[
ScaLAPACK ${SCALAPACK_VERSION}
===============

Built with:
- ${COMPILER}
- ${MPI}
- OpenBLAS ${OPENBLAS_VERSION}

Installed: $(date +"%d %B %Y")
]])

whatis("Name: ScaLAPACK")
whatis("Version: ${SCALAPACK_VERSION}")

local scalapack_root = "${SCALAPACK_ROOT}"

prepend_path("LD_LIBRARY_PATH",     pathJoin(scalapack_root, "lib"))
prepend_path("LIBRARY_PATH",        pathJoin(scalapack_root, "lib"))

setenv("SCALAPACK_ROOT", scalapack_root)
EOF
```

>Note: It is recommended to add "load(openblas, yourmpi)" into the modulefile to automatically load the dependencies.