For releasing the binary Ubuntu package of HSA-enabled clamp we repackaged the [HSA Okra runtime](https://github.com/HSAFoundation/Okra-Interface-to-HSA-Device) into a DEB that can be installed directly.

This is done thru the following dummy CMake/CPack configuration file that puts files required by CLAMP to /opt/hsa:


```
#!cmake

SET( CPACK_GENERATOR "DEB" )
SET(CPACK_SET_DESTDIR "ON") 
execute_process(COMMAND git describe --abbrev=4 --dirty --always --tags
                WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
                OUTPUT_VARIABLE GIT_VERSION
                OUTPUT_STRIP_TRAILING_WHITESPACE)
##### EXTERNAL LIB
INSTALL( FILES include/common.h include/okraContext.h DESTINATION include )
INSTALL( FILES bin/libamdhsacl64.so
               bin/libnewhsacore64.so
               bin/libokra_x86_64.so DESTINATION lib)
set(CPACK_SET_DESTDIR TRUE)
set(CPACK_INSTALL_PREFIX "/opt/hsa")
set(CPACK_PACKAGE_NAME "okra-dev")
set(CPACK_PACKAGE_VENDOR "HSA Foundation")
set(CPACK_PACKAGE_VERSION "0.1.1-${GIT_VERSION}")
set(CPACK_PACKAGE_VERSION_MAJOR "0")
set(CPACK_PACKAGE_VERSION_MINOR "1")
set(CPACK_PACKAGE_VERSION_PATCH "1")
set(CPACK_PACKAGE_FILE_NAME ${CPACK_PACKAGE_NAME}-${CPACK_PACKAGE_VERSION}-${CMAKE_SYSTEM_NAME})
set(CPACK_DEBIAN_PACKAGE_DESCRIPTION "OKRA is a runtime library that enables applications to do compute offloads to HSA-enabled GPUs")
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "Ray I-Jui Sung <ray@multicorewareinc.com>")
set(CPACK_DEBIAN_PACKAGE_DEPENDS "openjdk-7-jdk (>= 0.1.1)")
INCLUDE(CPack)
```

The packaged file (okra-dev-0.1.1-<git version>.deb) can be found at the download page, or [here](https://bitbucket.org/multicoreware/cppamp-driver-ng/downloads/okra-dev-0.1.1-4e62-Linux.deb)