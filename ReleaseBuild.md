```
#!bash

cmake .. -DCLANG_URL=https://ijsung@bitbucket.org/multicoreware/cppamp-ng.git -DLLVM_EXPERIMENTAL_TARGETS_TO_BUILD=CBackend -DGMAC_URL=https://bitbucket.org/multicoreware/gmac -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/opt/mcw
```