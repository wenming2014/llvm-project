# Flang

Flang is a ground-up implementation of a Fortran front end written in modern
C++. It started off as the f18 project (https://github.com/flang-compiler/f18)
with an aim to replace the previous flang project
(https://github.com/flang-compiler/flang) and address its various deficiencies.
F18 was subsequently accepted into the LLVM project and rechristened as Flang.

## Getting Started

Read more about flang in the [documentation directory](documentation).
Start with the [compiler overview](documentation/Overview.md).

To better understand Fortran as a language
and the specific grammar accepted by flang,
read [Fortran For C Programmers](documentation/FortranForCProgrammers.md)
and
flang's specifications of the [Fortran grammar](documentation/f2018-grammar.txt)
and
the [OpenMP grammar](documentation/OpenMP-4.5-grammar.txt).

Treatment of language extensions is covered
in [this document](documentation/Extensions.md).

To understand the compilers handling of intrinsics,
see the [discussion of intrinsics](documentation/Intrinsics.md).

To understand how a flang program communicates with libraries at runtime,
see the discussion of [runtime descriptors](documentation/RuntimeDescriptor.md).

If you're interested in contributing to the compiler,
read the [style guide](documentation/C++style.md)
and
also review [how flang uses modern C++ features](documentation/C++17.md).

## Supported C++ compilers

Flang is written in C++17.

The code has been compiled and tested with
GCC versions from 7.2.0 to 9.3.0.

The code has been compiled and tested with
clang version 7.0, 8.0, 9.0 and 10.0
using either GNU's libstdc++ or LLVM's libc++.

The code has been compiled on
AArch64, x86\_64 and ppc64le servers
with CentOS7, Ubuntu18.04, Rhel, MacOs, Mojave, XCode and
Apple Clang version 10.0.1.

The code does not compile with Windows and a compiler that does not have
support for C++17.

## Building Flang out of tree
These instructions are for building Flang separately from LLVM; if you are
building Flang alongside LLVM then follow the standard LLVM build instructions
and add flang to `LLVM_ENABLE_PROJECTS` instead, as detailed there.

### LLVM dependency

The instructions to build LLVM can be found at
https://llvm.org/docs/GettingStarted.html. If you are building flang as part
of LLVM, follow those instructions and add flang to `LLVM_ENABLE_PROJECTS`.

We highly recommend using the same compiler to compile both llvm and flang.

The flang CMakeList.txt file uses
the variable `LLVM_DIR` to find the installed LLVM components
and
the variable `MLIR_DIR` to find the installed MLIR components.

To get the correct LLVM and MLIR libraries included in your flang build,
define LLVM_DIR and MLIR_DIR on the cmake command line.
```
LLVM=<LLVM_BUILD_DIR>/lib/cmake/llvm \
MLIR=<LLVM_BUILD_DIR>/lib/cmake/mlir \
cmake -DLLVM_DIR=$LLVM -DMLIR_DIR=$MLIR ...
```
where `LLVM_BUILD_DIR` is
the top-level directory where LLVM was built.

### Building flang with GCC

By default,
cmake will search for g++ on your PATH.
The g++ version must be one of the supported versions
in order to build flang.

Or, cmake will use the variable CXX to find the C++ compiler. CXX should include
the full path to the compiler or a name that will be found on your PATH, e.g.
g++-8.3, assuming g++-8.3 is on your PATH.

```
export CXX=g++-8.3
```
or
```
CXX=/opt/gcc-8.3/bin/g++-8.3 cmake ...
```

### Building flang with clang

To build flang with clang,
cmake needs to know how to find clang++
and the GCC library and tools that were used to build clang++.

CXX should include the full path to clang++
or clang++ should be found on your PATH.
```
export CXX=clang++
```

### Installation Directory

To specify a custom install location,
add
`-DCMAKE_INSTALL_PREFIX=<INSTALL_PREFIX>`
to the cmake command
where `<INSTALL_PREFIX>`
is the path where flang should be installed.

### Build Types

To create a debug build,
add
`-DCMAKE_BUILD_TYPE=Debug`
to the cmake command.
Debug builds execute slowly.

To create a release build,
add
`-DCMAKE_BUILD_TYPE=Release`
to the cmake command.
Release builds execute quickly.

### Build Flang out of tree
```
cd ~/flang/build
cmake -DLLVM_DIR=$LLVM -DMLIR_DIR=$MLIR ~/flang/src
make
```
# How to Run Tests

Flang supports 2 different categories of tests
1. Regression tests (https://www.llvm.org/docs/TestingGuide.html#regression-tests)
2. Unit tests (https://www.llvm.org/docs/TestingGuide.html#unit-tests)

## For out of tree builds
To run all tests:
```
cd ~/flang/build
cmake -DLLVM_DIR=$LLVM -DMLIR_DIR=$MLIR ~/flang/src
make test check-all
```

To run individual regression tests llvm-lit needs to know the lit
configuration for flang. The parameters in charge of this are:
flang_site_config and flang_config. And they can be set as shown bellow:
```
<path-to-llvm-lit>/llvm-lit \
 --param flang_site_config=<path-to-flang-build>/test-lit/lit.site.cfg.py \
 --param flang_config=<path-to-flang-build>/test-lit/lit.cfg.py \
  <path-to-fortran-test>

```

Unit tests:

If flang was built with `-DFLANG_INCLUDE_TESTS=On` (`ON` by default), it is possible to generate unittests.
Note: Unit-tests will be skipped for LLVM install for an out-of-tree build as it does not include googletest related headers and libraries.

There are various ways to run unit-tests.

```

1. make check-flang-unit
2. make check-all or make check-flang
3. <path-to-llvm-lit>/llvm-lit \
        test/Unit
4. Invoking tests from <out-of-tree flang build>/unittests/<respective unit test folder>

```


## For in tree builds
If flang was built with `-DFLANG_INCLUDE_TESTS=On` (`On` by default), it is possible to
generate unittests.

To run all of the flang unit tests use the `check-flang-unit` target:
```
make check-flang-unit
```
To run all of the flang regression tests use the `check-flang` target:
```
make check-flang
```

# How to Generate Documentation

## Generate FIR Documentation
If flang was built with `-DLINK_WITH_FIR=On` (`On` by default), it is possible to
generate FIR language documentation by running `make flang-doc`. This will
create `docs/Dialect/FIRLangRef.md` in flang build directory.

## Generate Doxygen-based Documentation
To generate doxygen-style documentation from source code
- Pass `-DLLVM_ENABLE_DOXYGEN=ON -DFLANG_INCLUDE_DOCS=ON` to the cmake command.

```
cd ~/llvm-project/build
cmake -DLLVM_ENABLE_DOXYGEN=ON -DFLANG_INCLUDE_DOCS=ON ../llvm
make doxygen-flang

It will generate html in

    <build-dir>/tools/flang/docs/doxygen/html # for flang docs
```
