# fty-project
The goal of this project is to store common files used by the 42ity projects with the new cmake structure

## CMAKE
The cmake common settings will:

- include GNUInstallDirs variables 
- include CTest
- create a target memcheck which run the test with valgrind
- set differents warnings
- define macros (more information bellow on each macro)
- define the following options:
```
option(BUILD_TESTING "Build tests" OFF)
option(BUILD_DOC "Build documentation" OFF)
option(CREATE_PKGCONFIG "Create package config file" OFF)
option(CREATE_CMAKE_PKG "Create cmake packages" OFF)
```

To use it you have to add the following into your CMakeLists.txt file:

```
# Use common cmake settings
include(fty-project/cmake/fty-project.cmake)

```


### Dependency management
Dependency management is managed using the [ExternalProject_add](https://cmake.org/cmake/help/v3.13/module/ExternalProject.html) feature of `CMake`.

The [external](cmake/external) directory provides the cmake files to download and build dependencies.

To build the dependencies you need to use the cmake option "ENABLE_STANDALONE=ON"

### Cmake package
[cmakepackage.cmake](cmake/cmakepackage.cmake) define function to generate cmake package for library

The cmake option "CREATE_CMAKE_PKG=ON" must be specify to create and install a cmake package matching your target.

You can use the following macro to create the package:
  etn_create_lib_cmake_package(\<TARGET\> \<NAMESPACE\>)
  1. the name of your target
  1. the name of your namespace for cmake package. Each package must be in a namespace.

Example
```cmake
etn_create_lib_cmake_package("etn-pq-common" "EtnPQ")
```

The cmake package can be use by other cmake project to retrive PUBLIC compilation flags and PUBLIC dependencies.

### pkgconfig file
[pkgconfig.cmake](cmake/pkgconfig.cmake) define function to generate package config file library

The cmake option "CREATE_PKGCONFIG=ON" must be specify to create and install a package config file your target.

You can use the following macro to create the package:
  etn_create_lib_pkgconfig(\<TARGET\> [ \<CFLAGS\> ] [ \<LIBRARIES\> ])
  1. the name of your target
  1. list of public compilation flags (empty if no specific flags are needed)
  1. list of public dependencies (Retrieve from the target_link PUBLIC section if not specified)

Examples
```cmake
etn_create_lib_pkgconfig("etn-pq-common")
etn_create_lib_pkgconfig("etn-pq-common", "-Wno-error=narrowing -Wno-narrowing -std=c++17")
etn_create_lib_pkgconfig("etn-pq-common", "-Wno-error=narrowing -Wno-narrowing -std=c++17", "pthread dl")
etn_create_lib_pkgconfig("etn-pq-common", "", "pthread dl")
```

The pkgconfig file can be use by other cmake/autotools project to retrive PUBLIC compilation flags and PUBLIC dependencies.

### Target creation

Target has follow syntax:
```
etn_target([type] [target name] 
    SOURCES 
        [sources list] 
    USES 
        [private dependencies list] 
    USES_PUBLIC 
        [public dependencies]
    INCLUDE_DIRS 
        [include directories]
    PUBLIC 
        [public headers]
    PREPROCESSOR 
        [preprocessor definitions]
    FLAGS 
        [extra compilation flags]
    CMAKE 
        [extra cmake scripts]
    CONFIGS 
        [configs]
```

Where type could be:
 * `exe` - regular executable
 * `static` - static library
 * `shared` - shared library
 * `interface` - non binary library, just headers, configs etc

`USES` and `USES_PUBLIC` are dependencies of the project. Firstly system will try to find dependency in the system. 
If it will not found and ENABLE_STANDALONE is ON then will try to find it in `external` projects and will add it to compilation process.

Example of the project:
```
etn_target(exe ${PROJECT_NAME}
    SOURCES
        src/daemon.cpp
        src/include.hpp
    USES
        tntdb
)
```
