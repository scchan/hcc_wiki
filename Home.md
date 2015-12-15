# HCC : An open source C++ compiler for heterogeneous devices #
The HCC project is managed and primarily developed by [MulticoreWare](http://www.multicorewareinc.com/), a leading developer of heterogeneous software development platforms, tools and applications.  

****

# Download HCC #

### Download links for Ubuntu x86-64 packages ###
- [hcc](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/hcc-0.8.1549-ea9df54-27d8ed2-183de0b-Linux.deb)

### Download links for Ubuntu x86-64 tarballs ###
- [hcc](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/hcc-0.8.1549-ea9df54-27d8ed2-183de0b-Linux.tar.gz)

****

# Install dependencies #

On Ubuntu, make sure you have the following packages installed:

You can install all prerequisites with the command below:
```
sudo apt-get install cmake git subversion g++ libstdc++-4.8-dev libdwarf-dev libelf-dev libtinfo-dev libc6-dev-i386 gcc-multilib llvm llvm-dev llvm-runtime libc++1 libc++-dev libc++abi1 libc++abi-dev re2c libncurses5-dev
```

****

# Install HCC #

## Ubuntu binary packages for x86-64 ##

To install, download hcc DEB files from links above, and:

```
sudo dpkg -i hcc-<version>-Linux.deb
```

Default installation directory is /opt/hcc.

## Binary tarballs for x86-64 ##

To install, download hcc tar.gz files from links above, and:

```
sudo tar zxvf hcc-<version>-Linux.tar.gz
```

Default installation directory is /opt/hcc.

## Dynamic Libraries ##

Platform-specific libraries and libc++ are built as dynamic libraries.  After building, please change /etc/ld.so.conf to let dynamic libraries be locatable at runtime.

If you install deb files or tarballs, please add the following lines to /etc/ld.so.conf :
```
# HCC runtime libraries
/opt/hcc/lib
```

Please make sure OpenCL or HSA runtime libraries can be located by ld.so as well.  For example, your ld.so.conf might also need to include:
```
# OpenCL runtime (libOpenCL.so)
/opt/AMDAPP/lib/x86_64

# HSA runtime (libhsaruntime-64.so)
/opt/hsa/lib
```

Always remember to use: sudo ldconfig -v to reload ld.so cache.

****

# How to compile a C++AMP program #

Here's an example to build (compile + link) in 1-step:

```
# Assume HCC is installed and added to PATH
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

****

# API documentation

[API reference of HCC](http://whchung.bitbucket.org)
