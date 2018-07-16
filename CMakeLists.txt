cmake_minimum_required (VERSION 3.0)

# If available, enable developer warnings for the build system
if (POLICY CMP0005)
    cmake_policy(SET CMP0005 NEW)
endif ()

# include our cmake snippets
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

project(dbcsr)

include(CMakeDependentOption)

option(USE_MPI "Build with MPI support" ON)
option(WITH_EXAMPLES "Build the examples" ON)
cmake_dependent_option(WITH_C_API "Build the C API library" ON "USE_MPI" OFF) # the C API is only available when building with MPI

if (WITH_C_API AND WITH_EXAMPLES)
  set(ENABLED_LANGUAGES Fortran CXX)
else ()
  set(ENABLED_LANGUAGES Fortran)
endif ()

enable_language(${ENABLED_LANGUAGES})
enable_testing() # enables the `make test` target

# PACKAGE DISCOVERY:

find_package(BLAS REQUIRED)
find_package(LAPACK REQUIRED)

if (USE_MPI)
  find_package(MPI COMPONENTS ${ENABLED_LANGUAGES} REQUIRED)
endif ()

# make sure that the default is a RELEASE
if (NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE RELEASE CACHE STRING
    "Choose the type of build, options are: Debug Release."
    FORCE)
endif ()

# COMPILER CONFIGURATION:

get_filename_component(Fortran_COMPILER_NAME ${CMAKE_Fortran_COMPILER} NAME)

if (Fortran_COMPILER_NAME MATCHES "gfortran.*")
  set(CMAKE_CXX_FLAGS             "-fopenmp")
  set(CMAKE_CXX_FLAGS_RELEASE     "-O3 -funroll-loops")
  set(CMAKE_CXX_FLAGS_DEBUG       "-O0 -ggdb")
  set(CMAKE_Fortran_FLAGS         "-ffree-form -ffree-line-length-none -std=f2003 -fopenmp")
  set(CMAKE_Fortran_FLAGS_RELEASE "-O3 -funroll-loops")
  set(CMAKE_Fortran_FLAGS_DEBUG   "-O0 -ggdb")
  set(F2008_COMPILER_FLAGS        "-std=f2008ts")
elseif (Fortran_COMPILER_NAME MATCHES "ifort.*")
  set(CMAKE_CXX_FLAGS_RELEASE     "-O3")
  set(CMAKE_CXX_FLAGS_DEBUG       "-O0 -debug")
  set(CMAKE_Fortran_FLAGS         "-free -stand f03 -fpp")
  # Disable the line-length-extension warning #5268
  set(CMAKE_Fortran_FLAGS_RELEASE "-O3 -diag-disable=5268")
  set(CMAKE_Fortran_FLAGS_DEBUG   "-O0 -debug")
  set(F2008_COMPILER_FLAGS        "-stand f08")
else ()
  message("CMAKE_Fortran_COMPILER full path: " ${CMAKE_Fortran_COMPILER})
  message("Fortran compiler: " ${Fortran_COMPILER_NAME})
  message("No optimized Fortran compiler flags are known, we just try -O2...")
  set(CMAKE_Fortran_FLAGS_RELEASE "-O2")
  set(CMAKE_Fortran_FLAGS_DEBUG   "-O0 -g")
endif ()


add_subdirectory(src)
add_subdirectory(tests)

if (WITH_EXAMPLES)
  add_subdirectory(examples)
endif ()