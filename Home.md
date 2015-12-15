# HCC : An open source C++ compiler for heterogeneous devices #

The HCC project is managed and primarily developed by [MulticoreWare](http://www.multicorewareinc.com/), a leading developer of heterogeneous software development platforms, tools and applications.  

****

# Install HCC #

## Download HCC ##

- [hcc Ubuntu x86-64 package](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/hcc-0.8.1549-ea9df54-27d8ed2-183de0b-Linux.deb)

- [hcc Ubuntu x86-64 tarball](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/hcc-0.8.1549-ea9df54-27d8ed2-183de0b-Linux.tar.gz)

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

****

# API documentation #

[API reference of HCC](http://whchung.bitbucket.org)
