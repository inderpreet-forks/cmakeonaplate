# CMakeLists.txt
cmake_minimum_required(VERSION 3.0 FATAL_ERROR)
project(FooLib)

##############################################################################################################################################
# Standard CMake variables
##############################################################################################################################################

# BUILD_SHARED_LIBS is a standard CMake variable, but we declare it here to
# make it prominent in the GUI.
option(BUILD_SHARED_LIBS "Build shared libraries (DLLs)." OFF)

# Set a default build type if none was specified.
# See https://blog.kitware.com/cmake-and-the-default-build-type/
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  message(STATUS "Setting build type to 'Release' as none was specified.")
  set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build." FORCE)
  mark_as_advanced(CMAKE_BUILD_TYPE)
  # Set the possible values of build type for cmake-gui
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

# Export no symbols by default (if the compiler supports it). 
# This makes e.g. GCC's "visibility behavior" consistent with MSVC's.  
# On Windows/MSVC this is a noop. 
if (BUILD_SHARED_LIBS)
  set(CMAKE_C_VISIBILITY_PRESET hidden) 
  set(CMAKE_CXX_VISIBILITY_PRESET hidden) 
endif()

# Set the output folder where your program will be created
set(EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR}/bin)
set(   LIBRARY_OUTPUT_PATH ${CMAKE_BINARY_DIR}/bin)

##############################################################################################################################################
# Global settings
##############################################################################################################################################

# Product version according to Semantic Versioning v2.0.0 https://semver.org/
set(FOOLIB_VERSION_MAJOR 0)
set(FOOLIB_VERSION_MINOR 1)
set(FOOLIB_VERSION_PATCH 4)
set(FOOLIB_VERSION ${FOOLIB_VERSION_MAJOR}.${FOOLIB_VERSION_MINOR}.${FOOLIB_VERSION_PATCH})

# read license file
file(READ ${CMAKE_SOURCE_DIR}/../LICENSE.h LICENSE)

# version.h file
set(FOOLIB_VERSION_HEADER ${CMAKE_BINARY_DIR}/include/FooLib/version.h)
message("Generating ${FOOLIB_VERSION_HEADER}...")
configure_file( ${CMAKE_SOURCE_DIR}/src/FooLib/version.h.in ${FOOLIB_VERSION_HEADER} )

# config.h file
set(FOOLIB_CONFIG_HEADER ${CMAKE_BINARY_DIR}/include/FooLib/config.h)
message("Generating ${FOOLIB_CONFIG_HEADER}...")
if (BUILD_SHARED_LIBS)
  set(FOOLIB_BUILD_TYPE_CPP_DEFINE "#define FOOLIB_BUILT_AS_SHARED")
else()
  set(FOOLIB_BUILD_TYPE_CPP_DEFINE "#define FOOLIB_BUILT_AS_STATIC")
endif()
configure_file( ${CMAKE_SOURCE_DIR}/src/FooLib/config.h.in ${FOOLIB_CONFIG_HEADER} )
set(FOOLIB_BUILD_TYPE_CPP_DEFINE)

# Define installation directories
set(FOOLIB_INSTALL_BIN_DIR      "bin")
set(FOOLIB_INSTALL_LIB_DIR      "lib/FooLib-${FOOLIB_VERSION}")
set(FOOLIB_INSTALL_INCLUDE_DIR  "include/FooLib-${FOOLIB_VERSION}")
set(FOOLIB_INSTALL_CMAKE_DIR    ${FOOLIB_INSTALL_LIB_DIR}) # CMake files (*.cmake) should have the same destination as the library files. Some also prefers to use "cmake".

##############################################################################################################################################
# Project settings
##############################################################################################################################################

# Force a debug postfix if none specified.
# This allows publishing both release and debug binaries to the same location
# and it helps to prevent linking with the wrong library on Windows.
if(NOT CMAKE_DEBUG_POSTFIX)
  set(CMAKE_DEBUG_POSTFIX "-d")
endif()

# Prevents annoying warnings on MSVC
if (WIN32)
  add_definitions(-D_CRT_SECURE_NO_WARNINGS)
endif()

# Define include directories for source code.
# The specified values will not be exported.
set( FOOLIB_INCLUDE_DIR ${CMAKE_SOURCE_DIR}/include )
include_directories(${FOOLIB_INCLUDE_DIR}                 # public header files, for source code.
                    ${CMAKE_BINARY_DIR}/include/FooLib    # for ${FOOLIB_EXPORT_HEADER} and ${FOOLIB_CONFIG_HEADER} generated files.
)

# Subprojects
add_subdirectory(src/FooLib)

# unit tests
option(FOOLIB_BUILD_TEST "Build all FooLib's unit tests" OFF)
if(FOOLIB_BUILD_TEST)
  add_subdirectory(test)
endif()

##############################################################################################################################################
# Support for static and shared library
##############################################################################################################################################

if (BUILD_SHARED_LIBS)
  set(FOOLIB_EXPORT_HEADER_FILENAME "export.h")
  set(FOOLIB_EXPORT_HEADER ${CMAKE_BINARY_DIR}/include/FooLib/${FOOLIB_EXPORT_HEADER_FILENAME})
  message("Generating ${FOOLIB_EXPORT_HEADER_FILENAME} for shared library...")
  include (GenerateExportHeader) 
  GENERATE_EXPORT_HEADER(foolib 
               BASE_NAME foolib 
               EXPORT_MACRO_NAME FOOLIB_EXPORT 
               EXPORT_FILE_NAME ${FOOLIB_EXPORT_HEADER} 
               STATIC_DEFINE FOOLIB_BUILT_AS_STATIC
  )
endif()

##############################################################################################################################################
# Install
##############################################################################################################################################

# Install locations:   See https://unix.stackexchange.com/a/36874
#   On UNIX, installs to "/usr/local".
#   On Windows, installs to "C:\Program Files (x86)\${PROJECT_NAME}" or to "C:\Program Files\${PROJECT_NAME}" for 64 bit binaries

# Target config version verification file
configure_file(${CMAKE_SOURCE_DIR}/cmake/foolib-config-version.cmake.in ${CMAKE_CURRENT_BINARY_DIR}/cmake/foolib-config-version.cmake @ONLY)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/cmake/foolib-config-version.cmake DESTINATION ${FOOLIB_INSTALL_CMAKE_DIR})

# Target config file
configure_file(${CMAKE_SOURCE_DIR}/cmake/foolib-config.cmake.in ${CMAKE_CURRENT_BINARY_DIR}/cmake/foolib-config.cmake @ONLY)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/cmake/foolib-config.cmake DESTINATION ${FOOLIB_INSTALL_CMAKE_DIR})

install(DIRECTORY ${CMAKE_SOURCE_DIR}/include/FooLib DESTINATION ${FOOLIB_INSTALL_INCLUDE_DIR})
install(FILES ${FOOLIB_EXPORT_HEADER}
              ${FOOLIB_VERSION_HEADER}
              ${FOOLIB_CONFIG_HEADER}
              DESTINATION ${FOOLIB_INSTALL_INCLUDE_DIR}/FooLib)
install(EXPORT foolib-targets DESTINATION ${FOOLIB_INSTALL_CMAKE_DIR})

##############################################################################################################################################
# Packaging
##############################################################################################################################################

set(CPACK_PACKAGE_NAME ${PROJECT_NAME})
set(CPACK_PACKAGE_VERSION ${FOOLIB_VERSION})
set(CPACK_PACKAGE_VERSION_MAJOR "${FOOLIB_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${FOOLIB_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${FOOLIB_VERSION_PATCH}")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "CMakeOnaPlate - A CMake boilerplate for most C++ projects.")
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_SOURCE_DIR}/../LICENSE")
set(CPACK_RESOURCE_FILE_README "${CMAKE_SOURCE_DIR}/../README.md")

# we don't want to split our program up into several things
set(CPACK_MONOLITHIC_INSTALL 1)

# This must be last
include(CPack)
