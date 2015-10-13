# Overview #

This page is written to present how to integrate AMD Bolt in MCW CLAMP (C++ AMP based on OpenCL/HSA). The main purpose is to replace STL calls from user's application with Bolt AMP calls if any which are thought to be more efficient. And this replacement is initially expected to take place on AST level. In other way, we want it by clang frontend instead of IRs or any other transformation methods out of 'source-to-source' range.

***

# Goals #

The main design contains three parts as below. I also give the initial effort estimation in % out of total

1. Key issues in porting Bolt AMP code path  (Windows for now) onto MCW CLAMP path which lies on Linux system. (60%)
1. Adding clang plugin to invoke statement replacement in compilation stage. (39%)
1. Examples to showcase (1%)

***

# Why AMD Bolt? #

Bolt has nearly the same parlance as STL in its AMP code path. This includes API's function type: function name, arguments etc. which makes less efforts in statement replacement. Regarding this, other parallel libraries, or STL counterpart, generally need extra efforts to prepare additional arguments or different argument type. Another benefit is that porting Bolt AMP path can avoid difficulty in shifting a lot of STL conventions in order to make clang compiler happy.

***

# Key issues in porting Bolt AMP code path on Linux #

Before we go on this subsection, we should know about several key issues ahead.

1. MCW CLAMP needs gcc/g++ to build its compiler, viz clang++.  This means Bolt codes can not be directly distributed in MCW CLAMP code trunk since Bolt AMP codes include amp.h headers and are actually treated as MCW CLAMP application.
1. Bolt AMP codes needs to be built with MCW CLAMP clang++.  This means we need to make clang++ happy by shifting what are conformed with Microsoft MSVC to be conformed with clang++ on Linux.  This work can be more complicated since there are still some missing features of C++AMP specs. in MCW CLAMP
1. Whether Bolt AMP codes can be correctly emitted in MCW CLAMP GPU path is unknown.  This is most likely more difficult since we need extra efforts to correct MCW CLAMP implementations and more time to tests.

***

# Source descriptions #

```
SOURCE_DIR/lib/clang-plugins:  This folder contains implementations of clang plugin.

    EligibleSTLCallName.def :  definitions of eligible STL calls.
    ParallelRewriter.cpp:   Implementation of source codes rewriter borrowed from clang::Rewriter
    ParallelRewriter.h        Header
    StmtRewriter.cpp        Main Implementation to visit CallExpr and perform replacement of STL call with Bolt counterpart
    StmtRewriter.exports  Needed in linking plugin as it is a shared object

SOURCE_DIR/lib/clang-plugins/TestBolt:  This folder contains a sample to showcase two things:

SOURCE_DIR/tests/Unit/BoltRewrite/Transform_MaxElement.cpp : A sample program

SOURCE_DIR/Bolt:  This folder contains ported Bolt based on AMD Bolt v1.2
```

***
 
# Run Bolt AMP test cases #

If the test case do not show up in the folder mentioned above, viz. (BUILD_DIR)/Bolt/superbuild/Bolt-build/staging

Do the following to rebuild them:

```
# Go to (BUILD_DIR)/Bolt/superbuild/Bolt-build folder
make clean
CLAMP_NOTILECHECK=ON make
```

Status of each test.  Check Bolt/test/amp/CMakeLists.txt for the latest information.

```
# compile OK, failed some tests on HSA
add_subdirectory( BinarySearchTest )

# passed
add_subdirectory( PairTest )

# compile OK, failed some tests on HSA
add_subdirectory( GenerateTest )

# passed
add_subdirectory( FillTest )

# killed at opt within clamp-device
#add_subdirectory( InnerProductTest )

# passed
add_subdirectory( CopyTest )

# killed at opt within clamp-device
#add_subdirectory( MinElementTest )

# passed
add_subdirectory( MergeTest )

# killed at opt within clamp-device
#add_subdirectory( MaxElementTest )

# passed
add_subdirectory( CountTest )

# compile OK, crashed on HSA after passing some tests
add_subdirectory( PermutationIteratorTest )

# killed at opt within clamp-device
#add_subdirectory( ReduceTest )

# compile OK, crashed on HSA after passing some tests
add_subdirectory( ReduceByKeyTest )

# killed at opt within clamp-device
#add_subdirectory( ScanTest )

# killed at opt within clamp-device
#add_subdirectory( ScanByKeyTest )

# compile OK, crashed on HSA after passing some tests
add_subdirectory( SortTest )

# compile OK, failed some tests on HSA
add_subdirectory( SortByKeyTest )

# compile OK, crashed on HSA after passing some tests
add_subdirectory( StableSortTest )

# compile OK, failed some tests on HSA
add_subdirectory( StableSortByKeyTest )

# compile OK, crashed on HSA after passing some tests
add_subdirectory( TransformTest )

# killed at opt within clamp-device
#add_subdirectory( TransformReduceTest )

# killed at opt within clamp-device
#add_subdirectory( TransformScanTest )

# compile OK, crashed on HSA after passing some tests
add_subdirectory( ScatterTest )

# passed
add_subdirectory( GatherTest )

# compile OK, crashed on HSA after passing some tests
add_subdirectory( device_vector )
```


***
 
# Run the sample #

```
# go to (SOURCE_DIR)/tests/Unit/BoltRewrite
clamp-preprocess Transform_MaxElement.cpp test.cpp
```

You can see the following replacement in test.cpp has been done.

1. Bolt headers are added
1. std::max_element is rewritten as bolt::amp::max_element
1. std::transform is rewritten as bolt::amp::transform

```
# compile it
(BUILD_DIR)/compiler/bin/clang++ ` (BUILD_DIR)/build/Release/clamp-config --bolt --build --cxxflags --ldflags` -o test.out test.cpp

# run it
./test.out
```

The output would be:
```
Transform Neg - From Pointer
4    -4
0    0
5    -5
5    -5
0    0
5    -5
5    -5
1    -1
3    -3
1    -1
0    0
3    -3
1    -1
1    -1
3    -3
5    -5
Get max_element = 1000
100  <=  -122  <=  900  <=  1000
```

***
 
# Known issues #

1. if arguments are macros, we can select them to be expanded or not.  This is controlled by one of this plugin options. These options will be added later if needed.  This bug won't be fixed for now. There are poisonous libstdc++ linked in libLLVM*.  This feature can't be supported for now.
1. Need a more reliable way to identify "user source files" from "system source files"
1. Need more complicated test cases
1. Current Bolt implementation would fail tile uniformity check in C++AMP
 