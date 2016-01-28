# hcc: An Open-Source C++ Compiler for Heterogeneous Devices #
The hcc project (previously named Kalmar or Clamp) is managed and primarily developed by [MulticoreWare](http://www.multicorewareinc.com/), a leading vendor of heterogeneous software-development platforms, tools and applications. For more information, visit MulticoreWare's [C++ AMP software-development page](http://www.multicorewareinc.com/c-amp-software-development-services/).

# About hcc

This repository hosts hcc, a C++ compiler project. The goal is to implement a compiler front end that supports the following parallel-programming paradigms:

* C++ AMP 1.2
* [HC](HC mode), an HSA-specific extension to C++ AMP 1.2.
* C++17 parallel STL
* [OpenMP 3.1/4.0](OpenMP)

The front end would transform each type into HSAIL, SPIR binary or OpenCL-C. We have tested the following targets:

* Khronos OpenCL SPIR 1.2 and OpenCL C for
    * AMD stack/GPU with Khronos SPIR 1.2 and OpenCL C
    * Nvidia stack/GPU with OpenCL C
    * [Apple Mac OS X 10.9 stack](InstallOnMacOSX) with OpenCL C
* [HSAIL and BRIG](HSA Support Status) for these HSA devices:
    * AMD Kaveri APU
    * AMD Fiji dGPU
* AMD GCN (to be supplied)on
    * AMD Fiji dGPU

****

# API Documentation

See the [API reference for hcc](http://whchung.bitbucket.org).

****

## Compile and Install Dependencies ##

### Ubuntu

On Ubuntu, make sure you have installed the following packages:

* cmake
* git
* subversion
* g++
* libstdc++-4.8-dev
* libdwarf-dev
* libelf-dev
* libtinfo-dev
* libc6-dev-i386
* gcc-multilib
* llvm
* llvm-dev
* llvm-runtime
* libc++1
* libc++-dev
* libc++abi1
* libc++abi-dev
* re2c
* libncurses5-dev

To install all prerequisites, use the command below:
```
sudo apt-get install cmake git subversion g++ libstdc++-4.8-dev libdwarf-dev libelf-dev libtinfo-dev libc6-dev-i386 gcc-multilib llvm llvm-dev llvm-runtime libc++1 libc++-dev libc++abi1 libc++abi-dev re2c libncurses5-dev
```

You can now continue to build hcc from the source code or use prebuilt binary packages.

****

### CentOS

hcc currently depends on libc++ and libc++abi. CentOS lacks any libc++ binary packages, so you must build them manually. The OS will soon strip all dependencies on libc++ and libc++abi, so the procedure below will become simpler.

To build hcc on CentOS, first install the following prerequisite packages via yum:

* cmake
* git
* svn
* patch
* gcc
* gcc-c++
* epel-release

Install them using this command:

```
sudo yum install cmake git svn patch gcc gcc-c++ epel-release
```

After you have installed epel-release, install the other dependent packages:

* clang
* llvm-devel

To do so, use the command below:
```
sudo yum install clang llvm-devel
```

At this point, refer to the section “Build libc++ and libc++abi” below to finish building libc++ and libc++abi on CentOS.


****


### Fedora

hcc currently depends on libc++ and libc++abi. Fedora lacks any libc++ binary packages, so you must build them manually. Fedora will soon strip all dependencies on libc++ and libc++abi, so the procedure below will become simpler.

To build hcc on Fedora, first install the following prerequisite packages via yum:

* cmake
* git
* svn
* patch
* gcc
* gcc-c++
* clang
* llvm-devel
* re2c
* elfutils-libelf-devel

Install them using this command:

```
sudo yum install cmake git svn patch gcc gcc-c++ clang llvm-devel re2c elfutils-libelf-devel
```

At this point, refer to the section “Build libc++ and libc++abi” below to finish building libc++ and libc++abi on Fedora.

****

### Build libc++ and libc++abi

The following procedure is inspired by [a Stack Overflow post](http://stackoverflow.com/questions/25840088/how-to-build-libcxx-and-libcxxabi-by-clang-on-centos-7), with some modifications to ensure it works on both Fedora and CentOS.

First, get libc++ and build it:

```
# Get libcxx.
svn co http://llvm.org/svn/llvm-project/libcxx/trunk libcxx
cd libcxx
# We don’t recommend building libcxx in the source root directory.
# Instead, we make a build directory.
mkdir build
cd build
# Specifying CMAKE_BUILD_TYPE to Release generates performance-optimized code.
# Specify the absolute paths to Clang and Clang++ to CMAKE_C_COMPILER and DCMAKE_CXX_COMPILER,
# because CMake (ver. 2.8.12 - 3.0.x) has a bug--see http://www.cmake.org/Bug/view.php?id=15156.
# The CMAKE_INSTALL_PREFIX changes the install path from the default /usr/local to /usr.
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_C_COMPILER=/usr/bin/clang -DCMAKE_CXX_COMPILER=/usr/bin/clang++ ..
sudo make install
cd ../..
```

Next, get libc++abi and build it:

```
# Get libcxxabi.
svn co http://llvm.org/svn/llvm-project/libcxxabi/trunk libcxxabi
cd libcxxabi
mkdir build
cd build
# Without -DCMAKE_CXX_FLAGS="-std=c++11", Clang++ seems to use C++03, so libcxxabi (which is apparently written in C++11) won't compile. The problem could be a CMakeLists.txt bug in libcxxabi.
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_C_COMPILER=/usr/bin/clang -DCMAKE_CXX_COMPILER=/usr/bin/clang++ -DCMAKE_CXX_FLAGS="-std=c++11" -DLIBCXXABI_LIBCXX_INCLUDES=../../libcxx/include ..
sudo make install
cd ../..
```

Build libc++ again, this time with libc++abi support:

```
cd libcxx
mkdir build2
cd build2
# This time, we want to compile libcxx with libcxxabi, so we must specify LIBCXX_CXX_ABI=libcxxabi and the path to the libcxxabi headers, LIBCXX_CXX_ABI_INCLUDE_PATHS.
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_C_COMPILER=/usr/bin/clang -DCMAKE_CXX_COMPILER=/usr/bin/clang++ -DLIBCXX_CXX_ABI=libcxxabi -DLIBCXX_CXX_ABI_INCLUDE_PATHS=../../libcxxabi/include ..
sudo make install
cd ../..
```

You should now have succeeded in building libc++ and libc++abi and installing them on Fedora or CentOS. You can continue to build hcc from the source code.


****

# Building hcc From Source Code

To build hcc from the source code, follow these instructions:

```
mkdir hcc
cd hcc
git clone https://bitbucket.org/multicoreware/hcc.git src
mkdir build
cd build
# hcc will install to /opt/hcc by default
# change the target directory by running cmake -DCMAKE_INSTALL_PREFIX=<path where you want to install hcc>
cmake ../src
make -j4 world
make -j4
```

To build hcc in debug mode, perform these steps:

```
mkdir hcc
cd hcc
git clone https://bitbucket.org/multicoreware/hcc.git src
mkdir build
cd build
# hcc will install to /opt/hcc by default
# change the target directory by running cmake -DCMAKE_INSTALL_PREFIX=<path where you want to install hcc>
cmake -DCMAKE_BUILD_TYPE=Debug ../src
make -j1 world
make -j1
```

Note that the debug build requires more memory, so *make -j4* may fail, even on an 8GB machine. Thus, the debug build uses the command *make -j1.*

Once the build is complete, install it and set the necessary environmental variables:

```
# hcc will install to /opt/hcc by default
sudo make install

# set HCC_HOME environment
export HCC_HOME=/opt/hcc

# set PATH
export PATH=$HCC_HOME/bin:$PATH
```

If you want to build Ubuntu, Debian or a tar.gz archive for redistribution, invoke the following:

```
make package
```

****

# Downloads

[Download hcc](https://bitbucket.org/multicoreware/hcc/downloads/).

****

# Install #

## Ubuntu Binary Packages for x86-64 ##

To install hcc, download the deb files from the link above and run the following:

```
sudo dpkg -i hcc-<version>-Linux.deb
```

The default installation directory is /opt/hcc.

## Binary Tarballs for x86-64 ##

To install hcc, download the tar.gz files from link above and run the following:

```
sudo tar zxvf hcc-<version>-Linux.tar.gz
```

The default installation directory is /opt/hcc.

## Dynamic Libraries ##

Since hcc version 0.4.0, platform-specific libraries and libc++ are built as dynamic libraries. After you build them, change /etc/ld.so.conf to enable location of dynamic libraries at run time. If you install deb files or tarballs, add the following to /etc/ld.so.conf:
```
# hcc run-time libraries
/opt/hcc/lib
```

If you build from source code, add the following to /etc/ld.so.conf:
```
# hcc run-time implementations
(path_of_your_build_directory)/build/Release/lib
```

Ensure that ld.so can locate OpenCL and HSA run-time libraries as well. For example, your ld.so.conf might also need to include
```
# OpenCL run time (libOpenCL.so)
/opt/AMDAPP/lib/x86_64

# HSA run time (libhsaruntime-64.so)
/opt/hsa/lib
```

Always remember to use *sudo ldconfig –v* to reload the ld.so cache.

## Installing hcc on AMD Kaveri HSA ##

See the [HSA support status](HSA Support Status).

## Install on Mac OS X (Currently Obsolete) ##

See [InstallOnMacOSX](InstallOnMacOSX).


****

# Compiling C++ AMP Source Code #

The latest hcc release merges a new Clang driver to streamline the build process. Here's an example build (compile + link) in one step:

```
# Assume hcc is installed and added to PATH
# Use --install if you install hcc with an Ubuntu package
# Use --build if you build from source code
# If unspecified, the default is --install
clang++ `clamp-config --install --cxxflags --ldflags` -o test.out test.cpp
```

To use the HSA extension,
```
# Use -Xclang -fhsa-ext to enable HSA extension
clang++ `clamp-config --install --cxxflags --ldflags` -Xclang -fhsa-ext foo.cpp -o foo.out
```

The following command emits object files. Note that GPU codes will be stored in the special *.kernel* section.
```
# Use -c to emit object files
# GPU kernels will be stored in the .kernel section
clang++ `clamp-config --install --cxxflags` -c foo.cpp -o foo.o
```

Use this command to link objects---Clang will extract all *.kernel* sections from each object and lower to the target architecture (SPIR/OpenCL C/HSAIL):
```
# Clang will extract all .kernel sections from each object and lower to the target architecture (SPIR/OpenCL C/HSAIL)
clang++ `clamp-config --install --ldflags` foo.o bar.o -o foo.out
```

Since hcc version 0.5.0, you can generate CPU-only codes that need no GPU platform, such as OpenCL and HSA.
```
clang++ `clamp-config --install --cxxflags --ldflags` -cpu -o test.out test.cpp
```

Since version 0.6.0, you can check the hcc version for debugging purposes:
```
clang++ --version
```

Below is an example output:
```
hcc clang version 3.5.0 (tags/RELEASE_350/final) (based on hcc 0.8.1549-ea9df54-27d8ed2-183de0b LLVM 3.5.0svn)
Target: x86_64-unknown-linux-gnu
Thread model: posix

- 3.5.0 is the version of clang
- 0.8.1549 is the package version of hcc
- 1549 means the package is built on Week 49 of Year 2015
- ea9df54 is the git commit # of hcc driver
- 27d8ed2 is the git commit # of hcc compiler
- 183de0b is the git commit # of HLC HSAIL backend used by hcc
```

****

# Built-In hcc Macros

hcc includes these built-in macros:

| Macro | Meaning |
|----|--------|
| ```__HCC__``` | Always 1 |
| ```__hcc_major__``` | Major hcc version number |
| ```__hcc_minor__``` | Minor hcc version number |
| ```__hcc_patchlevel__``` | hcc patch level |
| ```__hcc_version__``` | Combined string containing ```__hcc_major__```, ```__hcc_minor__``` and ```__hcc_patchlevel__``` |

The rule for ```__hcc_patchlevel__``` is *yyWW-(hcc driver git commit #)-(hcc Clang git commit #).*
- *yy* stands for the last two digits of the year
- *WW* stands for the week number of the year

The following language-mode macros are available:

| Macro | Meaning |
|----|--------|
| ```__KALMAR_AMP__``` | 1 if in C++ AMP mode (*-std=c++amp*) |
| ```__KALMAR_HC__``` | 1 if in hc mode (*-hc*) |

# Compilation mode
hcc is a single-source compiler for which kernel codes and host codes can reside in the same file. Internally, it triggers two compilation iterations; user programs can employ the following macros to determine the compiler’s current mode.

| Macro | Meaning |
|----|--------|
| ```__KALMAR_ACCELERATOR__``` | Nonzero if the compiler is in kernel-code compilation mode |
| ```__KALMAR_CPU__``` | Nonzero if the compiler is in host-code compilation mode |

The values ```__KALMAR_ACCELERATOR__``` and ```__KALMAR_CPU__``` involve a few more details. Users seldom check these values.

- ```__KALMAR_ACCELERATOR__```---the macro value is 1 if the compiler is building for "normal" GPU targets such as OpenCL and HSA. The value is 2 when building kernels for execution on x86 (i.e., when using the *-cpu* option).
- ```__KALMAR_CPU__```---the macro value is 1 if the compiler is building host code for "normal" GPU targets such as OpenCL and HSA. The value is 2 when building host code for kernels that will execute on x86 (i.e., when using the *-cpu* option).

****

# Choose hcc Run Time #

hcc programs will automatically detect an available GPU platform of the following type:

- HSA
- OpenCL
- CPU

If you want to force hcc to employ a certain run time, use one of these approaches:
```
# force hcc run time to HSA
export HCC_RUNTIME=HSA

# force hcc run time to OpenCL
export HCC_RUNTIME=CL

# force hcc run time to CPU
export HCC_RUNTIME=CPU
```

To turn autodetection back on, use 
```
unset HCC_RUNTIME
```

Note that if hcc is unable to load the specified run time, it will fall back to automatic detection.

****

# SPIR vs. OpenCL C #

OpenCL machines with SPIR support will employ SPIR kernels instead of OpenCL C. You can alter the precedence using

```
# turn off SPIR, force use OpenCL C
export CLAMP_NOSPIR=1
```

The code above will force hcc to select OpenCL C instead of SPIR. To turn SPIR back on, use

```
# turn on SPIR
unset HCC_NOSPIR
```

The environment variable CLAMP_NOSPIR has no effect on devices that lack SPIR support.

****

# Sample Code #

We collected some [sample code](https://bitbucket.org/multicoreware/cppamp-sandbox) to illustrate hcc. This package is also available for [download](https://bitbucket.org/multicoreware/cppamp-sandbox/get/master.tar.gz). You must use the build script **buildme.binary** to correctly invoke the compiler and build C++ AMP code on Linux. See [README.BINARY.txt](https://bitbucket.org/multicoreware/cppamp-sandbox/src/master/README.BINARY.txt) for details.

****

# News

## 05/31/2015 ##

Maintenance release (0.6.0 release) 

### Changes ###
* Switched to Clang 3.5 as the default implementation
* Added HSA 1.0F support
* Fixed memory leaks in HSA
* Various bug fixes
* Preliminary C++17 Parallel STL implementation 

### Downloads for Ubuntu x86-64 Packages ###
- [clamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/kalmar-0.6.0-8769a29-c54a03c-Linux.deb)
- [libcxxamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/libcxxamp-0.6.0-8769-Linux.deb)
- [clamp-bolt](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/clamp-bolt-1.2.0-hsa-milestone4-Linux.deb) (optional)
- [boost](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/boost_1_55_0-hsa-milestone3.tar.gz) (optional)

### Downloads for Ubuntu x86-64 Tarballs ###
- [clamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/kalmar-0.6.0-8769a29-c54a03c-Linux.tar.gz)
- [libcxxamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/libcxxamp-0.6.0-8769-Linux.tar.gz)
- [clamp-bolt](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/clamp-bolt-1.2.0-hsa-milestone4-Linux.tar.gz) (optional)
- [boost](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/boost_1_55_0-hsa-milestone3.tar.gz) (optional)


## 02/01/2015 ##

HSA/OpenCL unified release (0.5.0 release, milestone 4)

### Changes ###
* Added HSA 1.0P support
* Fixed one major memory leak within Clang 3.3
* Various bug fixes
* Added more-generic support for SVM on HSA. You can now capture host objects using references in GPU kernels.
* Added preliminary support for "auto-auto" feature on HSA. GPU kernels need not necessarily carry the *restrict(amp)* specifier.
* Clang 3.5 support is mostly on par with Clang 3.3

### Downloads for Ubuntu x86-64 Packages ###
- [clamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/clamp-0.5.0-hsa-milestone4-Linux.deb)
- [libcxxamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/libcxxamp-0.5.0-hsa-milestone4-Linux.deb)
- [clamp-bolt](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/clamp-bolt-1.2.0-hsa-milestone4-Linux.deb) (optional)
- [boost](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/boost_1_55_0-hsa-milestone3.tar.gz) (optional)

### Clang 3.5 Upgrade ###

A new repository of Clang/LLVM 3.5 is available. To use it, follow these instructions to build from source code:

```
git clone https://bitbucket.org:/multicoreware/cppamp-driver-ng-35.git src
mkdir build
cd build
cmake ../src
make -j4 world && make
```

****

## 11/08/2014 ##

HSA/OpenCL unified release (0.4.0 release, milestone 3)

### Changes ###
* Unified HSA build and OpenCL build into one release package
* Simplified cmake procedure
* Introduced performance improvements in OpenCL and HSA
* Implemented “fat binary”: one C++ AMP binary can now contain multiple GPU-kernel versions (HSA/OpenCL)
* Decoupled C++ AMP programs from C++ AMP run times. You can now use an environment variable to dynamically pick which GPU platform to use.
* Implemented a preliminary port of AMD Bolt C++ AMP version from Windows to Linux
* Implemented a preliminary version of transforming C++ STL calls to AMD Bolt calls in order to accelerate normal C++ programs using a GPU

### Downloads for Ubuntu x86-64 Packages ###
- [clamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/clamp-0.4.0-hsa-milestone3-Linux.deb)
- [libcxxamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/libcxxamp-0.4.0-hsa-milestone3-Linux.deb)
- [clamp-bolt](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/clamp-bolt-1.2.0-hsa-milestone3-Linux.deb) (optional)
- [boost](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/boost_1_55_0-hsa-milestone3.tar.gz) (optional)

****

## 9/29/2014 ##

Updated OpenCL/SPIR release (0.3.0 release, milestone 2)

### Changes ###
* OpenCL/SPIR version based on the code base for second HSA release
* Improved Clang driver interface
* Implemented *restrict(auto),* an optional feature in C++ AMP Chapter 13

### Downloads for Ubuntu x86-64 (OpenCL/SPIR) Packages ###
- [clamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/clamp-0.3.0-hsa-milestone2-opencl-Linux.deb)
- [libcxxamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/libcxxamp-0.3.0-hsa-milestone2-opencl-Linux.deb)

****

## 9/27/2014 ##

Second HSA release (0.3.0 release, milestone 2)

### Changes ###
* Improved Clang driver interface
* Implemented *restrict(auto),* an optional feature in C++ AMP Chapter 13
* Relaxed C++ language rules on HSA
* Implemented new asynchronous parallel_for_each interface on HSA
* See [HSA Support Status](HSA Support Status) for more-detailed HSA-related information

### Downloads for Ubuntu x86-64 (HSA) Packages ###
- [clamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/clamp-0.3.0-hsa-milestone2-Linux.deb)
- [libcxxamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/libcxxamp-0.3.0-hsa-milestone2-Linux.deb)

****

## 8/18/2014 ##

First HSA release (0.3.0 release)

### Changes ###
* First C++ AMP release for HSA
* See [HSA Support Status](HSA Support Status) for more-detailed HSA-related information
* Note that the C++ AMP package for HSA is currently **incompatible** with OpenCL/SPIR
* Also note that C++ AMP for HSA **no longer** depends on the HSA Okra run time; the Okra port is no longer supported

### Downloads for Ubuntu x86-64 (HSA) Packages ###
- [clamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/clamp-0.3.0-hsa-milestone1-Linux.deb)
- [libcxxamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/libcxxamp-0.3.0-hsa-milestone1-Linux.deb)

***

## 7/2/2014 ##

Updated OpenCL/SPIR release

### Changes ###
* More bug fixes; conformance rate now greater than 99% on SPIR (passed + skipped)

### Downloads for Ubuntu x86-64 (OpenCL/SPIR) Packages ###
- [clamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/clamp-0.2.0-milestone5-135-g2580-Linux.tar.gz)
- [libcxxamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/libcxxamp-0.2.0-milestone5-135-g2580-Linux.tar.gz)

***

## 6/1/2014 ##
Milestone 5 (0.2.0 release)

### Changes ###
* Default installation directory changed to /opt/clamp
* Changed and encapsulated a few compile options; we suggest using *clamp-config* to abstract all compile options
* Various bug fixes thanks to MS conformance tests; conformance rate is now 97.5% on SPIR (passed + skipped)

### Downloads for Ubuntu x86-64 (OpenCL/SPIR) Packages ###
* [clamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/clamp-0.2.0-milestone5-Linux.deb)
* [libcxxamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/libcxxamp-0.2.0-milestone5-Linux.deb)

****

## 3/20/2014 ##
Milestone 4 (HSA/Okra port)

* HSA Foundation's HSAIL now using Okra run time for HSA devices (e.g., AMD Kaveri)
    * See the [latest HSAIL/Okra support status](HSAIL%20Support%20Status) for installation instructions and binary packages
* Note that building the HSA/Okra port requires a different configuration flag; this port is currently incompatible with the OpenCL/SPIR version

****

## 3/18/2014 ##
Milestone 3 updates (140 patches since milestone 3)

### Changes ###
* Various bug fixes thanks to MS conformance tests

****

## 3/15/2014 ##

A preliminary port to [HSA Okra run time](https://github.com/HSAFoundation/Okra-Interface-to-HSA-Device) is functional on a Kaveri machine. The most current status of HSAIL support is available [here](HSAIL%20Support%20Status).


****

# Older News #
See [here](OlderNews) for older news.

****

# Roadmap 

For a roadmap, see [here](Roadmap).

****

# More About This Project and Build Instructions

For more information about the project, see the [overview](https://bitbucket.org/multicoreware/hcc/overview).
