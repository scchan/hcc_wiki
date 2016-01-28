# HCC : An open source C++ compiler for heterogeneous devices #

This repository hosts HCC, a C++ compiler implementation project. The goal is to implement a compiler frontend which supports the following parallel programming paradigms:

* C++AMP 1.2
* [HC](HC mode), an HSA-specific extension to C++AMP 1.2.
* C++17 parallel STL
* [OpenMP 3.1/4.0](OpenMP)

and transforms it into HSAIL, SPIR binary, or OpenCL-C.

****

# Install HCC #

## Download HCC ##

- [hcc binary packages](https://bitbucket.org/multicoreware/hcc/downloads) : Ubuntu x86-64 debian package, or x86-64 .tar.gz tarballs are available.

## Install dependencies ##

You can install all prerequisites with the command below:
```
sudo apt-get install cmake git subversion g++ libstdc++-4.8-dev libdwarf-dev libelf-dev libtinfo-dev libc6-dev-i386 gcc-multilib llvm llvm-dev llvm-runtime libc++1 libc++-dev libc++abi1 libc++abi-dev re2c libncurses5-dev
```

## Install HCC ##

By default, HCC would be installed to /opt/hcc.

To install HCC, download hcc DEB files from links above, and then:
```
sudo dpkg -i hcc-<version>-Linux.deb
```

You can also choose to download hcc tar.gz files from links above, and then:

```
sudo tar zxvf hcc-<version>-Linux.tar.gz
```

## Setting up environment variables ##

Use the following command to set up environment variables needed for HCC and add it into PATH:

```
export HCC_HOME=/opt/hcc
export PATH=$HCC_HOME/bin:$PATH
```

## Setting up runtime libraries ##

HCC platform-specific runtimes are built as dynamic libraries.  Please change /etc/ld.so.conf to let dynamic libraries be locatable at runtime.

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

# How to use HCC #

Here's an example to build (compile + link) in 1-step:

To use C++ AMP mode:
```
# Assume HCC is installed and added to PATH
clang++ `clamp-config --cxxflags --ldflags` foo.cpp -o foo.out
```

To use HC mode:
```
# Assume HCC is installed and added to PATH
hcc `hcc-config --cxxflags --ldflags` foo.cpp -o foo.out
```
**Please notice HC is only available on HSA hardware at this moment. It doesn't support OpenCL.**

****

# API documentation #

[API reference of HCC](http://whchung.bitbucket.org)

****

# More information #

Please visit [Developer Information](Developer Information) for more detailed information.
