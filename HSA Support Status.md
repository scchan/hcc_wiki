This page summarizes the status of HSA support of Kalmar as well as build instructions.

# Dependencies #
 * Ubuntu 14.04 x86-64 (32-bit version has NOT been tested)
 * HSA Linux Driver at [https://github.com/HSAFoundation/HSA-Drivers-Linux-AMD]
 * HSA Runtime at [https://github.com/HSAFoundation/HSA-Runtime-AMD]

See section below about how to set up HSA development environment.

***

# Setting up HSA development environment #

To use Kalmar for HSA, please follow the steps below to make sure all dependencies are installed correctly.  In the future we hope HSA Foundation would publish Ubuntu packages to streamline this installation process.

## Setting up Ubuntu and HSA Linux driver ##
1. **(VERY IMPORTANT)** Uninstall AMD Catalyst Driver before doing any steps below!  Not even OpenCL 2.0 driver from AMD (14.41)!
1. Make sure Ubuntu 14.04 64-bit version or above has been installed.
1. Follow the instructions at [https://github.com/HSAFoundation/HSA-Drivers-Linux-AMD] to install HSA Linux driver Debian packages.
1. Invoke the command to configure kfd:
  ```
  echo "KERNEL==\"kfd\", MODE=\"0666\"" | sudo tee /etc/udev/rules.d/kfd.rules
  ```
1. Reboot.  Use "kfd_check_installation.sh" in HSA Linux driver to verify installation is correct.  Sample output in a correct installation should be:
```
Kaveri detected:............................Yes
Kaveri type supported:......................Yes
Radeon module is loaded:....................Yes
KFD module is loaded:.......................Yes
AMD IOMMU V2 module is loaded:..............Yes
KFD device exists:..........................Yes
KFD device has correct permissions:.........Yes
Valid GPU ID is detected:...................Yes

Can run HSA.................................YES
```

## Setting up HSA Runtime ##

* Download HSA Runtime at: [https://github.com/HSAFoundation/HSA-Runtime-AMD]:
```
git clone https://github.com/HSAFoundation/HSA-Runtime-AMD.git
```
* Follow the instructions at [https://github.com/HSAFoundation/HSA-Runtime-AMD] to install HSA Runtime Debian packages.

## (Optional) Setting up HSA HOF ##

* This is an optional tool to help you finalize HSA kernels at compile-time rather than runtime.  It can be found at: [https://github.com/HSAFoundation/HSA-HOF-AMD]
* Simply copy hof to /opt/amd/bin
```
sudo cp hof /opt/amd/bin
```
In case you want to install HOF under another directory, please refer to "cmake options" below to see how to manually specify the path when you build Kalmar.


## Compile and install other dependencies ##

Please follow steps described at [main page](https://bitbucket.org/multicoreware/hcc/wiki/Home).

***

# Build Kalmar for HSA #

## Build from Source ##

```
mkdir kalmar
cd kalmar
git clone https://bitbucket.org/multicoreware/hcc.git src

mkdir build
cd build
cmake ../src
make -j4 world
make -j4

# Now Kalmar compiler and runtimes will be built
# MAKE SURE you checked "dynamic libraries" section below to setup ld.so.conf for Kalmar runtime!

# (optional) test the build
make test

# (optional) test HSA-specific test cases
CLAMP_RUNTIME=HSA make test

# (optional) run C++AMP conformance test
make conformance-all
```

## cmake options ##

The default cmake options can be overridden on the command line.  For example:
```
# cmake ../src \
    -DCLANG_URL=https://bitbucket.org/multicoreware/cppamp-ng-35.git \
    -DHSA_HEADER_DIR=/opt/hsa/include \
    -DHSA_LIBRARY_DIR=/opt/hsa/lib \
    -DHSA_KMT_LIBRARY_DIR=/opt/hsa/lib \
    -DHSA_HOF_DIR=/opt/amd/bin \
    -DCMAKE_BUILD_TYPE=Release \
    -DCXXAMP_ENABLE_BOLT=OFF \
    -DHSAIL_COMPILER_DIR=/opt/amd/bin \
    -DHSAIL_ASSEMBLER_DIR=/opt/amd/bin \

```

For all paths below, please **USE ABSOLUTE PATH** to prevent any undefined behavior in cmake.

* CLANG_URL : URL of C++AMP clang implementation (default is https://bitbucket.org/multicoreware/cppamp-ng-35.git )
* HSA_HEADER_DIR : HSA runtime header include path (default is /opt/hsa/include )
* HSA_LIBRARY_DIR : HSA runtime library path (default is /opt/hsa/lib )
* HSA_KMT_LIBRARY_DIR : HSA kernel library path (default is /opt/hsa/lib )
* HSA_HOF_DIR : HSA offline finalizer binary path (default is /opt/amd/bin )
* CMAKE_BUILD_TYPE : Release or Debug (default is Release )
* CXXAMP_ENABLE_BOLT : ON / OFF AMD Bolt API (default is OFF )
* HSAIL_COMPILER_DIR : Place where HSAIL compiler (HLC) source code is placed. By default Kalmar compiler will checkout a fresh copy from github if this variable is not set.
* HSAIL_ASSEMBLER_DIR : Place where HSAIL-Tools (HSAILasm) source code is placed. By default Kalmar compiler will checkout a fresh copy from github if this variable is not set.

***

## Binary Packages of Kalmar for HSA ##
### Download links for Ubuntu x86-64 packages ###
- [kalmar](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/kalmar-0.6.0-8769a29-c54a03c-Linux.deb)
- [libcxxamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/libcxxamp-0.6.0-8769-Linux.deb)
- (optional) [clamp-bolt](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/clamp-bolt-1.2.0-hsa-milestone4-Linux.deb)
- (optional) [boost](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/boost_1_55_0-hsa-milestone3.tar.gz)

### Download links for Ubuntu x86-64 tarballs ###
- [kalmar](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/kalmar-0.6.0-8769a29-c54a03c-Linux.tar.gz)
- [libcxxamp](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/libcxxamp-0.6.0-8769-Linux.tar.gz)
- (optional) [clamp-bolt](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/clamp-bolt-1.2.0-hsa-milestone4-Linux.tar.gz)
- (optional) [boost](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/boost_1_55_0-hsa-milestone3.tar.gz)

See also [Downloads](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads) for older versions

****

# Install Kalmar for HSA #

## Ubuntu binary packages for x86-64 ##

To install, download Kalmar and libcxxamp DEB files from links above, and:

```
sudo dpkg -i libcxxamp-<version>-Linux.deb kalmar-<version>-Linux.deb
```

clamp-bolt and boost are optional packages if you want to use automatic replacement of C++ STL calls to AMD Bolt calls.
```
sudo dpkg -i clamp-bolt-<version>-Linux.deb
```

boost is only provided as a tarball.  Please use the following command to install it:
```
sudo tar zxvf boost_1_55_0-hsa-milestone3.tar.gz -C /opt/kalmar
```


Default installation directory is /opt/kalmar.

## Binary tarballs for x86-64 ##

To install, download kalmar and libcxxamp tar.gz files from links above, and:

```
sudo tar zxvf libcxxamp-<version>-Linux.tar.gz
sudo tar zxvf kalmar-<version>-Linux.tar.gz
```

clamp-bolt and boost are optional packages if you want to use automatic replacement of C++ STL calls to AMD Bolt calls.
```
sudo tar zxvf clamp-bolt-<version>-Linux.tar.gz
```

boost is only provided as a tarball.  Please use the following command to install it:
```
sudo tar zxvf boost_1_55_0-hsa-milestone3.tar.gz -C /opt/kalmar
```

Default installation directory is /opt/kalmar.

## Dynamic Libraries ##

Since 0.4.0 platform-specific libraries and libc++ are built as dynamic libraries.  After building, please change /etc/ld.so.conf to let dynamic libraries be locatable at runtime.

If you install deb files or tarballs, please add the following lines to /etc/ld.so.conf :
```
# Kalmar runtime libraries
# libc++ & Kalmar runtime implementations
/opt/kalmar/lib
```

If you build from source, please add the following lines to /etc/ld.so.conf:
```
# Kalmar runtime libraries
# libc++
(path_of_your_build_directory)/libc++/libcxx/lib
(path_of_your_build_directory)/libc++/libcxxrt/lib
# Kalmar runtime implementations
(path_of_your_build_directory)/build/Release/lib
```

Please make sure OpenCL or HSA runtime libraries can be located by ld.so as well.  For example, your ld.so.conf might also need to include:
```
# OpenCL runtime (libOpenCL.so)
/opt/AMDAPP/lib/x86_64

# HSA runtime (libhsa-runtime64.so and libhsa-runtime-ext64.so)
/opt/hsa/lib
```

Always remember to use: sudo ldconfig -v to reload ld.so cache.


***

# Use Kalmar for HSA #

## How to compile a C++ AMP source code ##

A new clang driver has been merged in the latest release to have a streamlined build process.  Here's an example to build (compile + link) in 1-step:

```
# Assume Kalmar and libcxxamp are installed and added to PATH
# use --install if you install Kalmar with ubuntu package
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

****

# Choose Kalmar runtime #

Kalmar programs will automatically detect available GPU platform on the system, with the following precendence:

- HSA
- OpenCL
- CPU

In case you want to force use a certain Kalmar runtime, you can use:
```
# force set runtime to HSA
export CLAMP_RUNTIME=HSA

# force set runtime to OpenCL
export CLAMP_RUNTIME=CL

# force set runtime to CPU
export CLAMP_RUNTIME=CPU
```

To turn auto detection on:
```
# turn on runtime auto detection 
unset CLAMP_RUNTIME
```

Please notice if Kalmar runtime find the specified runtime couldn't be loaded it would fall back to automatic detection.

***

# HSA Offline Finalization #

To save program startup time, kernels will be finalized at compile-time (offline finalization) if HOF [https://github.com/HSAFoundation/HSA-HOF-AMD] is available in the system.  This feature is currently only available if your build Kalmar from source.

At compile-time, Kalmar will check if HOF available and use it automatically.

At runtime, Kalmar runtime will pick offline finalized kernels first if they are available.  If there is no offline finalized kernels, BRIG kernels will then be picked and finalized.

To force use BRIG kernels instead of offline finalized ones, please use the environment variable CLAMP_NOISA:
```
# disable offline finalized kernels
export CLAMP_NOISA=ON
```

NOTE: Currently HOF will produce kernels for Kaveri only.  Kernels for Carrizo or Radeon R9 series will soon follow.

***

# HSA-specific extensions #

To exploit the full potential of HSA Architecture, Kalmar for HSA has several HSA-specific features implemented.  They are:

* Shared Virtual Memory : GPU and CPU share the same address space so it's possible to dereference memory on host within GPU kernels through raw pointers.
* Platform Atomics & Memory Order : C++11 atomic operations <atomic> are supported to synchronize GPU and CPU computations.  Memory orders can also be specified for fine-grained performance tuning.
* Dynamic Memory Allocation / Deallocation : In HSA kernels it's possible to allocate / deallocate host memory dynamically through C++ new / delete operators.
* Relax C++ language rules in AMP kernels : In HSA kernels we allow more C++ language rules previously forbidden in C++AMP specification.
* Asynchronous parallel_for_each : It's now possible to get a future object when you launch kernels and wait on its completion.  It's also possible to daisy chain kernel executions.
* Extend C++AMP restrict(auto) feature to semi-automatically deduce if a function (or even a loop) can be dispatched as a HSA kernel.


```
NOTICE:
Dynamic memory allocation / deallocation (Xmalloc version) is still **IN EXPERIMENTAL STAGE**.
**IT WILL BE CHANGED.  USE WITH CAUTION!**
You can use more stable malloc version with export ALWAYS_MALLOC=ON
```

For more detailed information about HSA-specific features, please refer to [HSA Features](HSA%20Features). 

***

# Conformance Status #

The result of Microsoft C++AMP Conformance Test Suite as of 01/19/2015:
```
==========================
 Passed:  2707 (98.868%)
 Skipped: 7 (0.256%)
 Failed:  24 (0.877%)
 Total:  2738
==========================
```

**Tip:** From my empirical experience literally all C++ AMP conformance test cases would finish within 30 seconds, so here's a script I use to terminate those test cases running too long on HSA.

```
#!/bin/bash
echo "Kill test.out which has been executed longer than 30 seconds"
while true; do
  sleep 3
  ps ux | awk '/test\.out$/ {split($10, a, ":"); if (a[2] > 30) print $2}' - | xargs kill -15
done
```