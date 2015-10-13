# 0.3.0 #

* Initial release on 8/18/2014.
* Second release on 9/27/2014.
* The next release would be on 11/08/2014.

### Feature (frontend) ###
* (done) Streamline compile process
* (done) Support optional features in C++ AMP
* (done) Improve double precision floating point support
* Upgrade to CLANG 3.5
* Integrate with AMD Bolt library
* Translate calls to STL algorithms to parallel ones

### Feature (backend) ###
* (done) Improve HSA support
* CPU-fallback in case of no GPU is present
* Single binary multiple GPU targets (OpenCL, HSA, CPU)
* Dynamic GPU target selection
* Support host targets other than x86-64

### Conformance ###
* (mostly done) Improve C++ AMP conformance rate on all supported targets (SPIR, OpenCL C, HSA)
* Test against more C++ AMP projects and libraries

***

# 0.2.0 #

* Initial release on 6/1/2014.

## Wish list ##

### Feature (frontend) ###
* Streamline compile process
* Support optional features in C++ AMP
* Improve double precision floating point support
* Upgrade to CLANG 3.4

### Feature (backend) ###
* Improve HSA support
* CPU-fallback in case of no GPU is present
* Single binary multiple GPU targets (OpenCL, HSA, CPU)
* Dynamic GPU target selection
* Support host targets other than x86-64

### Conformance ###
* Improve C++ AMP conformance rate on all supported targets (SPIR, OpenCL C, HSA)
* Test against more C++ AMP projects and libraries

****

# 0.1.1 #

## Milestone 5 ##

* Passing all applicable MS Conformance test

Released 6/1/2014.  Change to 0.2.0.

## Milestone 4 ##

* [HSAIL Support](HSAIL%20Support%20Status)

Released 3/20/2014

## Milestone 3 ##

* Statically deduce OpenCL address space
* Experimental SPIR 1.2 Code Generation

Released 2/6/2014

## Milestone 2 ##

* C++AMP Async support

Released 1/16/2014