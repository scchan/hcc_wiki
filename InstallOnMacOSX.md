This page describes how to install CLAMP on Mac OS X. The installation is based on Homebrew and should be fairly automated.

# Tested environments #

Note: may work on other version/hardware combinations but YMMV.
## OS X ##

MacOS X 10.9 Mavericks. Will not compile on Snow Leopard.

## Tested Hardware ##

* 2012 MacBook Air with Intel HD4000 GPU (Ivy Bridge).
* Early 2009 iMac with NVIDIA GeForce 9400 GPU.

## Note on C++AMP tile sizes on Intel HD4000 ##

The Mac OS X OpenCL stack on HD4000 has a maximum tile size of 256 threads (or OpenCL work-items), which is _smaller_ than what required by C++AMP 1.2 spec, or 1024, as stated in line 3007 of the specification. So some samples are changed to make sure their tile sizes work on Macs with HD4000.

# Install Homebrew #

Install [HomeBrew](http://brew.sh/) 

# Install CLAMP #

brew install http://raw.github.com/ijsung/homebrew/master/Library/Formula/clamp.rb --HEAD

Note, --HEAD is necessary as currently the last tagged version (milestone3 as of now) used in default clamp.rb doesn't build on MacOS X. Also make sure that you have installed xcode command line tool.

## Troubleshooting build failures ##

brew install --verbose --debug http://raw.github.com/ijsung/homebrew/master/Library/Formula/clamp.rb --HEAD

Note: ensure that you have proper permission to folders in /usr/local/. Additionally, if you encounter errors installing wget, install it manually

# Run Regression Test #

TBD: write test script driver for Homebrew

Currently, you will need to use build script **buildme.binary** in the [Samples](https://bitbucket.org/multicoreware/cppamp-sandbox) [(download)](https://bitbucket.org/multicoreware/cppamp-sandbox/get/master.tar.gz) to correctly invoke the compiler and build C++AMP code. Follow the steps listed in "To Compile" section in [README.BINARY.TXT](https://bitbucket.org/multicoreware/cppamp-sandbox/src/master/README.BINARY.txt) for details.

# Note when using buildme.binary script in cppamp-sandbox #

Change the line

PREFIX=/opt/mcw/

to

PREFIX=/usr/local