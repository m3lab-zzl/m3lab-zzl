# 在M1电脑上安装vasp6

部分操作需提前完成科学上网配置

## 1. 安装xcode和homebrew

在终端中输入以下命令：

```bash
xcode-select --install
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

## 2. 安装依赖库

在终端中输入以下命令：

```bash
brew install gcc openmpi scalapack fftw qd openblas
```

## 3. 修改 makefile.include

解压 vasp6.2.0.tar.gz，进入vasp.6.2.0文件夹，基于 makefile.include.gnu_omp 修改 makefile.include 文件：

```bash
cd vasp.6.2.0
cp arch/makefile.include.gnu_omp makefile.include
```

makefile.include 文件内容如下:

```bash
# Default precompiler options
CPP_OPTIONS= -DHOST=\"LinuxGNU\" \
              -DMPI -DMPI_BLOCK=8000 \
              -Duse_collective \
              -DscaLAPACK \
              -DCACHE_SIZE=4000 \
              -Davoidalloc \
              -Dvasp6 \
              -Duse_bse_te \
              -Dtbdyn \
              -Dfock_dblbuf \
              -D_OPENMP \
              -Dqd_emulate

CPP        = gcc-13 -E -P -C -w $*$(FUFFIX) >$*$(SUFFIX) $(CPP_OPTIONS)

FC         = mpif90 -fopenmp
FCL        = mpif90 -fopenmp

FREE       = -ffree-form -ffree-line-length-none

FFLAGS      = -w -ffpe-summary=invalid,zero,overflow -L /opt/homebrew/Cellar/gcc/13.2.0/lib/gcc/13
OFLAG      = -O2
OFLAG_IN   = $(OFLAG)
DEBUG      = -O0

OBJECTS    = fftmpiw.o fftmpi_map.o  fftw3d.o  fft3dlib.o
OBJECTS_O1 += fftw3d.o fftmpi.o fftmpiw.o
OBJECTS_O2 += fft3dlib.o

# For what used to be vasp.5.lib
CPP_LIB    = $(CPP)
FC_LIB     = $(FC)
CC_LIB     = gcc-13
CFLAGS_LIB = -O
FFLAGS_LIB = -O1
FREE_LIB   = $(FREE)

OBJECTS_LIB= linpack_double.o

# For the parser library
CXX_PARS   = g++-13
LLIBS      =  -lstdc++
QD         ?= /opt/homebrew
LLIBS      += -L$(QD)/lib -lqdmod -lqd
INCS       += -I$(QD)/include/qd

# When compiling on the target machine itself, change this to the
# relevant target when cross-compiling for another architecture
FFLAGS     += -march=native

# For gcc-10 and higher (comment out for older versions)
FFLAGS     += -fallow-argument-mismatch

# BLAS and LAPACK (mandatory)
OPENBLAS_ROOT ?= /opt/homebrew/Cellar/openblas/0.3.23
BLASPACK    = -L$(OPENBLAS_ROOT)/lib -lopenblas

# scaLAPACK (mandatory)
SCALAPACK_ROOT ?= /opt/homebrew
SCALAPACK   = -L$(SCALAPACK_ROOT)/lib -lscalapack

LLIBS      += $(SCALAPACK) $(BLASPACK)

# FFTW (mandatory)
FFTW_ROOT  ?= /opt/homebrew
LLIBS      += -L$(FFTW_ROOT)/lib -lfftw3 -lfftw3_omp
INCS       += -I$(FFTW_ROOT)/include

# HDF5-support (optional but strongly recommended)
#CPP_OPTIONS+= -DVASP_HDF5
#HDF5_ROOT  ?= /path/to/your/hdf5/installation
#LLIBS      += -L$(HDF5_ROOT)/lib -lhdf5_fortran
#INCS       += -I$(HDF5_ROOT)/include
```

必须修改部分内容：

```sh
ls /opt/homebrew/bin/gcc* # 检查homebrew安装的gcc大版本，例如gcc-13；
ls /opt/homebrew/Cellar/gcc/ # 检查gcc小版本号，例如13.2.0
# 据此修改CPP，第一个FFLAG, CC_LIB, CXX_PARS，参数中的gcc版本

ls /opt/homebrew/Cellar/openblas/ # 检查openblas版本，例如0.3.23
# 据此修改OPENBLAS_ROOT参数
```

## 4. 编译vasp并测试

在终端中输入以下命令：

```bash
make veryclean
make std gam ncl
export OMP_NUM_THREADS=1 # 单线程，否则会与多进程冲突
make test
```
