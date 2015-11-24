# HCC : An open source C++ compiler for heterogeneous devices #
The HCC project (previously named as Kalmar, Clamp) is managed and primarily developed by [MulticoreWare](http://www.multicorewareinc.com/), a leading developer of heterogeneous software development platforms, tools and applications.  
More information: MulticoreWare's [C++ AMP software development](http://www.multicorewareinc.com/c-amp-software-development-services/).

# About HCC

This repository hosts HCC, a C++ compiler implementation project. The goal is to implement a compiler frontend which supports the following parallel programming paradigms:

* C++AMP 1.2
* [HC](HC mode), an HSA-specific extension to C++AMP 1.2.
* C++17 parallel STL
* [OpenMP 4.0](OpenMP)

and transforms it into HSAIL, SPIR binary, or OpenCL-C.

Tested targets are:

* Khronos OpenCL SPIR 1.2 and OpenCL C for
    * AMD Stack/AMD GPU with Khronos SPIR 1.2 and OpenCL C
    * NVIDIA Stack/NVIDIA GPU with OpenCL C
    * [Apple Mac OS X 10.9 Stack](InstallOnMacOSX) with OpenCL C
* [HSAIL and BRIG](HSA Support Status) for HSA devices:
    * AMD Kaveri APU
    * AMD Fiji dGPU
* AMD GCN (TO BE SUPPLIED)
    * AMD Fiji dGPU

****

# API documentation

[API reference of HCC](http://whchung.bitbucket.org)

****

## Compile and install dependencies ##

### Ubuntu

On Ubuntu, make sure you have the following packages installed:

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

You can install all prerequisites with the command below:
```
sudo apt-get install cmake git subversion g++ libstdc++-4.8-dev libdwarf-dev libelf-dev libtinfo-dev libc6-dev-i386 gcc-multilib llvm llvm-dev llvm-runtime libc++1 libc++-dev libc++abi1 libc++abi-dev re2c libncurses5-dev
```

You are able to continue build hcc from source, or use pre-built binary packages now.

****

### CentOS

hcc is currently dependent on libc++ and libc++abi. On CentOS, there are no libc++ binary packages, so they have to be built manually.

In the near future, all dependencies to libc++ and libc++abi will be stripped so steps here would become simpler.

To build hcc on CentOS, first install prerequisite packages via yum:

* cmake
* git
* svn
* patch
* gcc
* gcc-c++
* epel-release

You can install these prerequisites with the command below:

```
sudo yum install cmake git svn patch gcc gcc-c++ epel-release
```

After epel-release is installed, install other dependent packages:

* clang
* llvm-devel

You can install these prerequisites with the command below:
```
sudo yum install clang llvm-devel
```

Once you reach here, please refer to "Build libc++ and libc++abi" below to finish building libc++ and libc++abi on CentOS.


****


### Fedora

hcc is currently dependent on libc++ and libc++abi. On Fedora, there are no libc++ binary packages, so they have to be built manually.

In the near future, all dependencies to libc++ and libc++abi will be stripped so steps here would become simpler.

To build hcc on Fedora, first install prerequisite packages via yum:

* cmake
* git
* svn
* patch
* gcc
* gcc-c++
* clang
* llvm-devel
* re2c

You can install these prerequisites with the command below:

```
sudo yum install cmake git svn patch gcc gcc-c++ clang llvm-devel re2c
```

Once you reach here, please refer to "Build libc++ and libc++abi" below to finish building libc++ and libc++abi on Fedora.

****

### Build libc++ and libc++abi

Steps here are inspired by [this stackoverflow post](http://stackoverflow.com/questions/25840088/how-to-build-libcxx-and-libcxxabi-by-clang-on-centos-7), with some modifications to ensure they work on both Fedora and CentOS.

- Get libc++ and build it

```
# Get libcxx.
svn co http://llvm.org/svn/llvm-project/libcxx/trunk libcxx
cd libcxx
# It is not recommended to build libcxx in the source root directory.
# So, we make a build directory.
mkdir build
cd build
# Specifying CMAKE_BUILD_TYPE to Release shall generate performance optimized code.
# Please specify the absolute paths to clang and clang++ to CMAKE_C_COMPILER and DCMAKE_CXX_COMPILER,
# because CMake (ver. 2.8.12 - 3.0.x) has a bug ... See http://www.cmake.org/Bug/view.php?id=15156
# The CMAKE_INSTALL_PREFIX changes the install path from the default /usr/local to /usr.
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_C_COMPILER=/usr/bin/clang -DCMAKE_CXX_COMPILER=/usr/bin/clang++ ..
sudo make install
cd ../..
```

- Get libc++abi and build it

```
# Get libcxxabi.
svn co http://llvm.org/svn/llvm-project/libcxxabi/trunk libcxxabi
cd libcxxabi
mkdir build
cd build
# Without -DCMAKE_CXX_FLAGS="-std=c++11", clang++ seems to use c++03, so libcxxabi which seems to be written in C++11 can't be compiled. It could be a CMakeLists.txt bug of libcxxabi.
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_C_COMPILER=/usr/bin/clang -DCMAKE_CXX_COMPILER=/usr/bin/clang++ -DCMAKE_CXX_FLAGS="-std=c++11" -DLIBCXXABI_LIBCXX_INCLUDES=../../libcxx/include ..
sudo make install
cd ../..
```

- Build libc++ again, this time with libc++abi support

```
cd libcxx
mkdir build2
cd build2
# This time, we want to compile libcxx with libcxxabi, so we have to specify LIBCXX_CXX_ABI=libcxxabi and the path to libcxxabi headers, LIBCXX_CXX_ABI_INCLUDE_PATHS.
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_C_COMPILER=/usr/bin/clang -DCMAKE_CXX_COMPILER=/usr/bin/clang++ -DLIBCXX_CXX_ABI=libcxxabi -DLIBCXX_CXX_ABI_INCLUDE_PATHS=../../libcxxabi/include ..
sudo make install
cd ../..
```

By now libc++ and libc++abi should be built and installed on your Fedora/CentOS installation. You are able to continue build hcc from source now.


****

# Build hcc from source

Please use the following instructions to build from source as of now:

```
mkdir hcc
cd hcc
git clone https://bitbucket.org/multicoreware/cppamp-driver-ng-35.git src
mkdir build
cd build
# hcc will be installed to /opt/hcc by default
# change it by running cmake -DCMAKE_INSTALL_PREFIX=<path you want to install hcc>
cmake ../src
make -j4 world
make -j4
```

Once the build is completed, install it and set necessary environmental variables:

```
# hcc will be installed to /opt/hcc by default
sudo make install

# set HCC_HOME environment
export HCC_HOME=/opt/hcc

# set PATH
export PATH=$HCC_HOME/bin:$PATH
```

****

# Downloads

In the latest release (0.7.0), OpenCL and HSA are unified into one single release package.

### Download links for Ubuntu x86-64 packages ###
- [clamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/kalmar-0.6.0-f69d9d4-02e68c9-Linux.deb)
- [libcxxamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/libcxxamp-0.6.0-f69d-Linux.deb)
- (optional) [clamp-bolt](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/clamp-bolt-1.2.0-hsa-milestone4-Linux.deb)
- (optional) [boost](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/boost_1_55_0-hsa-milestone3.tar.gz)

### Download links for Ubuntu x86-64 tarballs ###
- [clamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/kalmar-0.6.0-f69d9d4-02e68c9-Linux.tar.gz)
- [libcxxamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/libcxxamp-0.6.0-f69d-Linux.tar.gz)
- (optional) [clamp-bolt](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/clamp-bolt-1.2.0-hsa-milestone4-Linux.tar.gz)
- (optional) [boost](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/boost_1_55_0-hsa-milestone3.tar.gz)

See also [Downloads](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads) for older versions

****

# Install #

## Ubuntu binary packages for x86-64 ##

To install, download clamp and libcxxamp DEB files from links above, and:

```
sudo dpkg -i libcxxamp-<version>-Linux.deb kalmar-<version>-Linux.deb
```

Default installation directory is /opt/kalmar.

## Binary tarballs for x86-64 ##

To install, download kalmar and libcxxamp tar.gz files from links above, and:

```
sudo tar zxvf libcxxamp-<version>-Linux.tar.gz
sudo tar zxvf kalmar-<version>-Linux.tar.gz
```

Default installation directory is /opt/kalmar.

## Dynamic Libraries ##

Since 0.4.0 platform-specific libraries and libc++ are built as dynamic libraries.  After building, please change /etc/ld.so.conf to let dynamic libraries be locatable at runtime.

If you install deb files or tarballs, please add the following lines to /etc/ld.so.conf :
```
# C++AMP runtime libraries
# libc++ & C++AMP runtime implementations
/opt/kalmar/lib
```

If you build from source, please add the following lines to /etc/ld.so.conf:
```
# C++AMP runtime implementations
(path_of_your_build_directory)/build/Release/lib
```

Please make sure OpenCL or HSA runtime libraries can be located by ld.so as well.  For example, your ld.so.conf might also need to include:
```
# OpenCL runtime (libOpenCL.so)
/opt/AMDAPP/lib/x86_64

# HSA runtime (libhsaruntime-64.so)
/opt/hsa/lib
```

Always remember to use: sudo ldconfig -v to reload ld.so cache.

## Install on Mac OS X (Experimental) ##

See [InstallOnMacOSX](InstallOnMacOSX)

## Install on AMD Kaveri HSA (Experimental) ##

See [HSA Support Status](HSA Support Status)

****

# How to compile a C++ AMP source code #

A new clang driver has been merged in the latest release to have a streamlined build process.  Here's an example to build (compile + link) in 1-step:

```
# Assume HCC and libcxxamp are installed and added to PATH
# use --install if you install HCC with ubuntu package
# use --build if you build from source
# if not specified, default would be --install
clang++ `clamp-config --install --cxxflags --ldflags` -o test.out test.cpp
```

To use HSA-extension:
```
# Use -Xclang -fhsa-ext to enable HSA extension
clang++ `clamp-config --install --cxxflags --ldflags` -Xclang -fhsa-ext foo.cpp -o foo.out
```

To emit object files.  Please notice GPU codes will be stored in a special section ".kernel".
```
# Use -c to emit object files.
# GPU kernels will be stored in .kernel section
clang++ `clamp-config --install --cxxflags` -c foo.cpp -o foo.o
```

To link objects.  Clang will extract all ".kernel" sections from each objects and lower to target architecture (SPIR/OpenCL C/HSAIL)
```
# clang will extract all .kernel sections from each objects and lower to target architecture (SPIR/OpenCL C/HSAIL)
clang++ `clamp-config --install --ldflags` foo.o bar.o -o foo.out
```

Since 0.5.0, it is also possible to generate CPU-only codes which don't need any GPU platforms such as OpenCL or HSA.
```
clang++ `clamp-config --install --cxxflags --ldflags` -cpu -o test.out test.cpp
```

Since 0.6.0, it's possible to check HCC version for debugging purpose:
```
clang++ --version
```

Example output would be:
```
HCC clang version 3.5.0 (tags/RELEASE_350/final) (based on HCC 0.6.0-8769a29-c54a03c LLVM 3.5.0svn)
Target: x86_64-unknown-linux-gnu
Thread model: posix

- 3.5.0 is the version of clang
- 0.6.0 is the package version of HCC
- 8769a29 is the git commit # of HCC driver
- c54a03c is the git commit # of HCC compiler
```

****

# HCC built-in macros

Built-in macros:

| Macro | Meaning |
|----|--------|
| ```__HCC__``` | always be 1 |
| ```__hcc_major__``` | major version number of HCC |
| ```__hcc_minor__``` | minor version number of HCC |
| ```__hcc_patchlevel__``` | patchlevel of HCC |
| ```__hcc_version__``` | combined string of ```__hcc_major__```, ```__hcc_minor__```, ```__hcc_patchlevel__``` |

The rule for ```__hcc_patchlevel__``` is: yyWW-(HCC driver git commit #)-(HCC clang git commit #)
- yy stands for the last 2 digits of the year
- WW stands for the week number of the year

Macros for language modes in use:

| Macro | Meaning |
|----|--------|
| ```__KALMAR_AMP__``` | 1 in case in C++ AMP mode (-std=c++amp) |
| ```__KALMAR_HC__``` | 1 in case in HC mode (-hc) |

Compilation mode:
HCC is a single-source compiler where kernel codes and host codes can reside in the same file. Internally HCC would trigger 2 compilation iterations, and the following macros can be user by user programs to determine which mode the compiler is in.

| Macro | Meaning |
|----|--------|
| ```__KALMAR_ACCELERATOR__``` | not 0 in case the compiler runs in kernel code compilation mode |
| ```__KALMAR_CPU__``` | not 0 in case the compiler runs in host code compilation mode |

There are a few more details about the values ```__KALMAR_ACCELERATOR__``` and ```__KALMAR_CPU__```.  Normally users aren't expected to check the values of them.

- ```__KALMAR_ACCELERATOR__``` : The macro has value 1 in case the compiler is building for "normal" GPU targets such as OpenCL or HSA.  The value is 2 if we are building kernels for execution on x86 (ie. when you use -cpu option).
- ```__KALMAR_CPU__``` : The macro has value 1 in case the compiler is building host codes for "normal" GPU targets such as OpenCL or HSA.  The value is 2 if we are building host codes for kernels to be executed on x86 (ie. when you use -cpu option).

****

# Choose C++AMP runtime #

C++AMP programs will automatically detect available GPU platform on the system, with the following precendence:

- HSA
- OpenCL
- CPU

In case you want to force C++AMP runtime to use a certain runtime, you can use:
```
# force set C++AMP runtime to HSA
export CLAMP_RUNTIME=HSA

# force set C++AMP runtime to OpenCL
export CLAMP_RUNTIME=CL

# force set C++AMP runtime to CPU
export CLAMP_RUNTIME=CPU
```

To turn auto detection back on:
```
unset CLAMP_RUNTIME
```

Please notice if C++AMP runtime find the specified runtime couldn't be loaded it would fall back to automatic detection.

****

# SPIR v. OpenCL C #

On OpenCL machines with SPIR support, SPIR kernels will be used instead of OpenCL C ones.  You can alter the precedence by:

```
# turn off SPIR, force use OpenCL C
export CLAMP_NOSPIR=1
```

It will force C++AMP runtime to pick OpenCL C instead of SPIR.  To turn it back on:

```
# turn on SPIR
unset CLAMP_NOSPIR
```

The environment variable CLAMP_NOSPIR has no effect on devices without SPIR support.

****

# Sample codes #

We have collected a few [sample codes](https://bitbucket.org/multicoreware/cppamp-sandbox).  The package is also available for [download](https://bitbucket.org/multicoreware/cppamp-sandbox/get/master.tar.gz).

You will need to use the build script **buildme.binary** to correctly invoke the compiler and build C++AMP codes on Linux. See [README.BINARY.TXT](https://bitbucket.org/multicoreware/cppamp-sandbox/src/master/README.BINARY.txt) for details.

****

# News

## 05/31/2015 ##

Maintenance Release (0.6.0 Release)

### Changes ###
* Switch to Clang 3.5 as the default implementation
* Support HSA 1.0F
* Fix memory leaks in HSA
* Various bug fixes
* Preliminary C++17 Parallel STL implementation 

### Download links for Ubuntu x86-64 packages ###
- [clamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/kalmar-0.6.0-8769a29-c54a03c-Linux.deb)
- [libcxxamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/libcxxamp-0.6.0-8769-Linux.deb)
- (optional) [clamp-bolt](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/clamp-bolt-1.2.0-hsa-milestone4-Linux.deb)
- (optional) [boost](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/boost_1_55_0-hsa-milestone3.tar.gz)

### Download links for Ubuntu x86-64 tarballs ###
- [clamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/kalmar-0.6.0-8769a29-c54a03c-Linux.tar.gz)
- [libcxxamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/libcxxamp-0.6.0-8769-Linux.tar.gz)
- (optional) [clamp-bolt](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/clamp-bolt-1.2.0-hsa-milestone4-Linux.tar.gz)
- (optional) [boost](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/boost_1_55_0-hsa-milestone3.tar.gz)


## 02/01/2015 ##

HSA/OpenCL Unified Release (0.5.0 Release Milestone 4)

### Changes ###
* Support HSA 1.0P
* Fix one major memory leak within Clang 3.3
* Various bug fixes
* Support more generic SVM on HSA.  It is now possible to capture host objects by reference in GPU kernels.
* Preliminary support "auto-auto" feature on HSA.  GPU kernels do not necessarily have to carry restrict(amp) specifier.
* Clang 3.5 support is mostly on par with Clang 3.3

### Download links for Ubuntu x86-64 packages ###
- [clamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/clamp-0.5.0-hsa-milestone4-Linux.deb)
- [libcxxamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/libcxxamp-0.5.0-hsa-milestone4-Linux.deb)
- (optional) [clamp-bolt](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/clamp-bolt-1.2.0-hsa-milestone4-Linux.deb)
- (optional) [boost](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/boost_1_55_0-hsa-milestone3.tar.gz)

### Clang 3.5 upgrade ###

A new repository of Clang/LLVM 3.5 is available.  To use it, please use the following instructions to build from source as of now:

```
git clone https://bitbucket.org:/multicoreware/cppamp-driver-ng-35.git src
mkdir build
cd build
cmake ../src
make -j4 world && make
```

****

## 11/08/2014 ##

HSA/OpenCL Unified Release (0.4.0 Release Milestone 3)

### Changes ###
* Unified HSA build and OpenCL build into one release package
* Simplified cmake procedure.
* Introduced performance improvements in OpenCL and HSA.
* Implemented "fat binary" : one C++AMP binary could now contain multiple versions of GPU kernels (HSA / OpenCL)
* Decoupled C++AMP programs from C++AMP runtimes.  It's now possible to use an environment variable to dynamically pick which GPU platform to use.
* Implemented a preliminary port of AMD Bolt C++AMP version from Windows to Linux.
* Implemented a preliminary version of transforming C++ STL calls to AMD Bolt calls, in order to make normal C++ programs be accelerated by GPU.

### Download links for Ubuntu x86-64 packages ###
- [clamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/clamp-0.4.0-hsa-milestone3-Linux.deb)
- [libcxxamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/libcxxamp-0.4.0-hsa-milestone3-Linux.deb)
- (optional) [clamp-bolt](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/clamp-bolt-1.2.0-hsa-milestone3-Linux.deb)
- (optional) [boost](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/boost_1_55_0-hsa-milestone3.tar.gz)

****

## 9/29/2014 ##

Update OpenCL/SPIR Relase (0.3.0 Release Milestone 2)

### Changes ###
* OpenCL/SPIR version based on the codebase for Second HSA Release
* Improved clang driver interface
* Implemented restrict(auto) which is an optional feature in C++AMP Chapter 13.

### Download links for Ubuntu x86-64 (OpenCL/SPIR) packages ###
- [clamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/clamp-0.3.0-hsa-milestone2-opencl-Linux.deb)
- [libcxxamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/libcxxamp-0.3.0-hsa-milestone2-opencl-Linux.deb)

****

## 9/27/2014 ##

Second HSA Rlease (0.3.0 Release Milestone 2)

### Changes ###
* Improved clang driver interface
* Implemented restrict(auto) which is an optional feature in C++AMP Chapter 13.
* Relaxed C++ language rules on HSA.
* Implemented new asynchronous parallel_for_each interface on HSA.
* See [HSA Support Status](HSA Support Status) for more detailed HSA-related information.

### Download links for Ubuntu x86-64 (HSA) packages ###
- [clamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/clamp-0.3.0-hsa-milestone2-Linux.deb)
- [libcxxamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/libcxxamp-0.3.0-hsa-milestone2-Linux.deb)

****

## 8/18/2014 ##

First HSA Release (0.3.0 Release)

### Changes ###
* First C++AMP for HSA release.
* See [HSA Support Status](HSA Support Status) for more detailed HSA-related information.
* Please notice C++AMP for HSA package is NOT compatible with OpenCL/SPIR at this moment.
* Please also notice C++AMP for HSA does NOT depend on HSA Okra runtime anymore, and Okra port would NOT be supported anymore.

### Download links for Ubuntu x86-64 (HSA) packages ###
- [clamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/clamp-0.3.0-hsa-milestone1-Linux.deb)
- [libcxxamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/libcxxamp-0.3.0-hsa-milestone1-Linux.deb)

***

## 7/2/2014 ##

Updated OpenCL/SPIR Release

### Changes ###
* More bug fixes.  Conformance rate is more than 99% on SPIR now.  (passed + skipped).

### Download links for Ubuntu x86-64 (OpenCL/SPIR) packages ###
- [clamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/clamp-0.2.0-milestone5-135-g2580-Linux.tar.gz)
- [libcxxamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/libcxxamp-0.2.0-milestone5-135-g2580-Linux.tar.gz)

***

## 6/1/2014 ##
Milestone 5 (0.2.0 Release)

### Changes ###
* Default installation directory changed to /opt/clamp .
* Changed and encapsulated a few compile options.  We suggest to use clamp-config to abstract away all compile options.
* Various bug fixes thanks to MS Conformance Tests.  Now we have 97.5% conformance rate on SPIR (passed + skipped).

### Download links for Ubuntu x86-64 (OpenCL/SPIR) packages ###
* [clamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/clamp-0.2.0-milestone5-Linux.deb)
* [libcxxamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/libcxxamp-0.2.0-milestone5-Linux.deb)

****

## 3/20/2014 ##
Milestone 4 (HSA/Okra port)

* HSA Foundation's HSAIL using Okra runtime for HSA devices (e.g. AMD Kaveri).
    * See the [latest status of HSAIL/Okra support](HSAIL%20Support%20Status) for installation instructions and binary packages.
* Note that the HSA/Okra port requires a different configuration flag to build and is currently not compatible with the OpenCL/SPIR version.

****

## 3/18/2014 ##
Milestone 3 Updates (140 patches since milestone 3)

### Changes ###
* Various bug fixes thanks to MS Conformance Tests

****

## 3/15/2014 ##

A preliminary port to [HSA Okra runtime](https://github.com/HSAFoundation/Okra-Interface-to-HSA-Device) is functional on a Kaveri machine. The most current status of HSAIL support can be found at [here](HSAIL%20Support%20Status)


****

# Older News #
[Here](OlderNews)

****

# Roadmap 

See [here](Roadmap)

****

# More about this project and build instruction

See [the Overview](https://bitbucket.org/multicoreware/cppamp-driver-ng/overview)