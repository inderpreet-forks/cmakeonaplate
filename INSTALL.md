# Install #

The FooLib's project defines a specific target to install the compiled binaries on the current system.

To install the compiled binaries, run the following commands according the your platform:

## Windows ##

On the Windows platform, run the following command:
```cmake
cmake --build . --config <config> --target INSTALL
```

Note: the default installation path is `C:\Program Files (x86)\FooLib`.

## Linux ##

On the Linux platform, run the following command:
```cmake
make install
```
Note: the default installation path is `/usr/local`.



### Installation file structure ###

When executing the `install` target, the following files will be installed on a *Windows* system:

```
C:\PROGRAM FILES (X86)\FOOLIB
+---bin
|       footest.exe
|
+---include
|   \---FooLib-0.1.3
|       \---FooLib
|               config.h
|               export.h
|               foolib.h
|               version.h
|
\---lib
    \---FooLib-0.1.3
            foolib-config-version.cmake
            foolib-config.cmake
            foolib-targets-release.cmake
            foolib-targets.cmake
            foolib.lib
```

Note that the file structure above is generated by building the FooLib library as a `shared library` in `Release` mode.


# Build #

This section explains how to compile and build the software and how to get a development environment ready.



## Prerequisites ##

### Software Requirements ###
The following software must be installed on the system for compiling source code:

* [Google C++ Testing Framework v1.8.0](https://github.com/google/googletest/tree/release-1.8.0) (untested with other versions)
* [CMake](http://www.cmake.org/) v3.4.3 (or newer)

### Linux Requirements ###

These are the base requirements to build and use FooLib:

  * GNU-compatible Make or gmake
  * POSIX-standard shell
  * A C++98-standard-compliant compiler

### Windows Requirements ###

* Microsoft Visual C++ 2010 or newer



## Build steps ##

The following steps show how to build the library:

1) Configure the _Visual Studio solution_ or the _Makefile_ using the following commands:

```cmake
mkdir build
cd build
cmake ../library
```

2) Build the source code:
   1) On Windows, run `cmake --build . --config Release` or open `FooLib.sln` with Visual Studio.
   2) On Linux, run `make` command.



### Build options ###

The following table shows the available build option supported:

| Name | Type | Default | Usage |
|------|------|:-------:|-------|
| CMAKE_INSTALL_PREFIX | STRING | See CMake documentation | Defines the installation folder of the library.           |
| FOOLIB_BUILD_TEST    | BOOL   | OFF                     | Enable/disable the generation of unit tests target.       |
| BUILD_SHARED_LIBS    | BOOL   | OFF                     | Enable/disable the generation of shared library makefiles |

To enable a build option, run the following command at the cmake configuration time:
```cmake
cmake -D<BUILD-OPTION-NAME>=ON ..
```
