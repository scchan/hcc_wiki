# Build Kalmar with Grid Launch Support #
There is a feature branch named grid_launch_integration that allows building of Grid Launch code.
### Starting with fresh build:
```
git clone git@bitbucket.org:multicoreware/cppamp-driver-ng-35.git $kalmar_src
cd $kalmar_src && git checkout grid_launch_integration
```
Run `git-show` to make sure it's on commit `# c865cd2...`

You must also checkout the branch under clang. If this is a fresh clone of Kalmar, you must run cmake first to fetch clang
```
cd ..
mkdir $kalmar_build
cd $kalmar_build
cmake ../$kalmar_src

cd $kalmar_src/compiler/tools/clang
git checkout grid_launch_integration
```
Run `git-show` to make sure it's on commit `# 29b33ed...`

### Continue with build:
```
cd $kalmar_build
make world -j4 && make -j4
```

### Run tests:
```
export CLAMP_RUNTIME=HSA
make test
```
make test should report close to the following:
```
  Expected Passes    : 453
  Expected Failures  : 16
  Unsupported Tests  : 11
  Unexpected Failures: 18
```
```
make conformance
```
All tests before the very last one should pass. The following may fail:
```
$kalmar_src/amp-conformance/Tests/7_para_for_each/ComputeDomain/extent_max/test.cpp : failed, timeout
```

***
### Running individual Grid Launch code:

To try out the current individual test case look at the following directory:  
```
$kalmar_src/tests/Unit/HIP  
file: vectoradd_hc.cpp
```
If Kalmar's clang and clamp-config are in your path, the procedure for compiling the grid launch code is the same as compiling Kalmar code while adding a -hc flag to the command:

Replace ```--build``` with ```--install``` if you're using an installed version of Kalmar.   
```clang `clamp-config --build --cxxflags --ldflags` -hc vectoradd_hc.cpp -o vectoradd```

Separate linkage also works:  
```
clang `clamp-config --build --cxxflags` -hc vectoradd_hc.cpp -o vectoradd.o
clang `clamp-config --build --ldflags` -hc vectoradd.o -o vectoradd
```

Run the executable!  
```
./vectoradd
```  
Error code will return 0 and nothing should appear if the executable ran correctly.

***
Possible Issues:  
A recent change in Kalmar incorporated HLC and HSAILasm into the build process. You might need to install additional dependencies as stated in the updated [documentation](https://bitbucket.org/multicoreware/cppamp-driver-ng/wiki/Home). 