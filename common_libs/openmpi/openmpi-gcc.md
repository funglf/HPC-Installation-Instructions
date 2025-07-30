# OpenMPI Installation Guide


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
DEPENDENCIES=${COMPILER}
```

### If building for CUDA aware MPI
```bash
CUDA_VERSION=$(nvcc --version | grep "release" | sed -E 's/.*release ([0-9]+\.[0-9]+).*/\1/')
CUDA=cuda${CUDA_VERSION}
DEPENDENCIES=${COMPILER}-${CUDA}
```
>When building CUDA aware MPI, I am assuming you are on HPC cluster hence I will be using NVHPC.

## Build UCX/UCC
Please Visit Guide on ucx and ucc.

## Build Libevent 
```bash 
# Set Version and Directory for installation
LIBEVENT_VERSION=2.1.12
LIBEVENT_ROOT=${SW_DIR}/libevent/${LIBEVENT_VERSION}-${COMPILER}

# Download source code
cd ${BUILD_DIR}
wget https://github.com/libevent/libevent/releases/download/release-${LIBEVENT_VERSION}-stable/libevent-${LIBEVENT_VERSION}-stable.tar.gz

tar -xvf libevent-${LIBEVENT_VERSION}-stable.tar.gz
cd libevent-${LIBEVENT_VERSION}-stable

# configure
./autogen.sh
./configure --prefix=$LIBEVENT_ROOT

# build and install
make -j 8
make install
```

## Build HWLOC [(hardware locality)](https://www.open-mpi.org/projects/hwloc/)
```bash
HWLOC_VERSION=2.12.0
HWLOC_ROOT=${SW_DIR}/hwloc/${HWLOC_VERSION}-${DEPENDENCIES}

cd ${BUILD_DIR}
HWLOC=hwloc-${HWLOC_VERSION}
wget https://download.open-mpi.org/release/hwloc/v2.12/${HWLOC}.tar.gz

tar -xvf ${HWLOC}.tar.gz
cd ${HWLOC}

mkdir build
cd build

../configure --prefix=${HWLOC_ROOT}
```
**If with CUDA:**
```bash
../configure --prefix=${HWLOC_ROOT} --with-cuda=${NVHPC_ROOT}/cuda
```

```bash 
make -j 8 
make install
```

## Build PMIX
```bash
PMIX_VERSION=5.0.7
PMIX_ROOT=${SW_DIR}/pmix/${PMIX_VERSION}-${COMPILER}

cd ${BUILD_DIR}
PMIX=pmix-$PMIX_VERSION
wget https://github.com/openpmix/openpmix/releases/download/v${PMIX_VERSION}/${PMIX}.tar.gz

tar -xvf ${PMIX}.tar.gz 
cd ${PMIX}

mkdir build
cd build


../configure CFLAGS="-I${HWLOC_ROOT}/include" LDFLAGS="-L${HWLOC_ROOT}/lib" \
                            --with-hwloc=${HWLOC_ROOT} \
                            --with-libevent=${LIBEVENT_ROOT} \
                            --prefix=${PMIX_ROOT} \
                            --with-zlib

make -j 8 
make install
```

## Build PRRTE
```bash
PRRTE_VERSION=3.0.9
PRRTE_ROOT=${SW_DIR}/prrte/${PRRTE_VERSION}-${COMPILER}

cd ${BUILD_DIR}
PRRTE=prrte-${PRRTE_VERSION}
wget https://github.com/openpmix/prrte/releases/download/v${PRRTE_VERSION}/${PRRTE}.tar.gz

tar -xvf ${PRRTE}.tar.gz
cd ${PRRTE}

./autogen.pl
mkdir build && cd build

../configure --with-hwloc=${HWLOC_ROOT} \
                --with-libevent=${LIBEVENT_ROOT} \
                --with-pmix=${PMIX_ROOT} \
                --prefix=${PRRTE_ROOT}

make -j 8 
make install
```

## Build OpenMPI CPU only

### Define UCX/UCC directories 
```bash
UCX_ROOT= #where you installed ucx 
UCC_ROOT= #where you installed ucc
```
If you are not using them, please take out the options down below that involves them.

### Configure
```bash
CFLAGS="-I${HWLOC_ROOT}/include -I${LIBEVENT_ROOT}/include -I${UCC_ROOT}/include -I${UCX_ROOT}/include"
LDFLAGS="-L${HWLOC_ROOT}/lib -L${LIBEVENT_ROOT} -L${UCC_ROOT}/lib -L${UCX_ROOT}/lib"

CONF="CC=${CC} CXX=${CXX} FC=${FC} "\
"--enable-mpi1-compatibility "\
"--enable-mpi-fortran "\
"--enable-mpi-interface-warning "\
"--enable-mpirun-prefix-by-default "\
"--with-ucx=${UCX_ROOT} "\
"--with-ucc=${UCC_ROOT} "\
"--with-zlib "\
"--with-libevent=${LIBEVENT_ROOT} "\
"--with-hwloc=${HWLOC_ROOT} "\
"--with-pmix=${PMIX_ROOT} "\
"--with-prrte=${PRRTE_ROOT} "\

# print CONF
echo "CONF: $CONF"

../configure CFLAGS="${CFLAGS}" LDFLAGS="${LDFLAGS}" $CONF --prefix=${OPENMPI_ROOT}
```

>For HPC installation:
>
>Resource Manager are often used such as slurm. The option for `slurm` needs to be passed into the configure step.
>
> Parallel Filesystem are also often used. The option for `lustre` or equivalent needs to be passed into the configure step.

### Build
```bash
make -j 8

make install
```

## Build OpenMPI CUDA Aware
### Define UCX/UCC directories -- needs to be CUDA AWARE 
```bash
UCX_ROOT= #where you installed ucx 
UCC_ROOT= #where you installed ucc
```
If you are not using them, please take out the options down below that involves them.

### Define SHARP/HCOLL directories 
SHARP and HCOLL are libaries within the nvidia communication software packages [DOCA](https://developer.nvidia.com/networking/doca) and [HPCX](https://developer.nvidia.com/networking/hpc-x) that offloads collective communications


```bash
HCOLL_ROOT=/opt/mellanox/hcoll
SHARP_ROOT=/opt/mellanox/sharp
```
If you are not using them, please take out the options down below that involves them.


### Configure
```bash
CFLAGS="-I${HWLOC_ROOT}/include -I${LIBEVENT_ROOT}/include -I${HCOLL_ROOT}/include -I${SHARP_ROOT}/include -I${UCC_ROOT}/include -I${UCX_ROOT}/include"
LDFLAGS="-L${HWLOC_ROOT}/lib -L${LIBEVENT_ROOT} -L${HCOLL_ROOT}/lib -L${SHARP_ROOT}/lib -L${UCC_ROOT}/lib -L${UCX_ROOT}/lib"


CONF="CC=${CC} CXX=${CXX} FC=${FC} "\
"--enable-mpi1-compatibility "\
"--enable-mpi-fortran "\
"--enable-mpi-interface-warning "\
"--enable-mpirun-prefix-by-default "\
"--with-ucx=${UCX_ROOT} "\
"--with-ucc=${UCC_ROOT} "\
"--with-ucc-libdir=${UCC_ROOT}/lib "\
"--with-libevent=${LIBEVENT_ROOT} "\
"--with-hwloc=${HWLOC_ROOT} "\
"--with-pmix=${PMIX_ROOT} "\
"--with-prrte=${PRRTE_ROOT} "\
"--with-cuda=${NVHPC_ROOT}/cuda/${CUDA_VERSION} "\
"--with-cuda-libdir=${NVHPC_ROOT}/cuda/${CUDA_VERSION}/lib64/stubs "\
"--with-fca "\
"--with-zlib "\
"--with-hcoll=${HCOLL_ROOT} "\
"--with-sharp=${SHARP_ROOT} "

# print CONF
echo "CONF: $CONF"

../configure CFLAGS="${CFLAGS}" LDFLAGS="${LDFLAGS}" $CONF --prefix=${OPENMPI_ROOT}
```

>For HPC installation:
>
>Resource Manager are often used such as slurm. The option for `slurm` needs to be passed into the configure step.
>
> Parallel Filesystem are also often used. The option for `lustre` or equivalent needs to be passed into the configure step.

> Note on CUDA-AWARE installation
> When including CUDA, for version 5.0.x, the configure command must be with:
> ```bash
>--with-cuda-libdir=${NVHPC_ROOT}/cuda/${CUDA_VERSION}/lib64/stubs #dont forget /stubs
>```


### Build
```bash
make -j 8

make install
```
