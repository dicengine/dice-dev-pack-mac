cmake_minimum_required(VERSION 3.4)
MESSAGE(STATUS "DICe dev-pack for mac")

PROJECT(DICe::DevPack::Mac)

MESSAGE(STATUS "Using LAPACK from: ${LAPACK}")
MESSAGE(STATUS "Using BLAS from: ${BLAS}")
MESSAGE(STATUS "Using C compiler: ${CMAKE_C_COMPILER}")
MESSAGE(STATUS "Using C++ compiler: ${CMAKE_CXX_COMPILER}")
MESSAGE(STATUS "Using Fortran compiler: ${CMAKE_Fortran_COMPILER}")
if(NOT DEFINED BUILD_TRILINOS)
  set(BUILD_TRILINOS ON)
endif()
MESSAGE(STATUS "Build Trilinos: ${BUILD_TRILINOS}")
if(NOT DEFINED BUILD_OPENCV)
  set(BUILD_OPENCV ON)
endif()
MESSAGE(STATUS "Build OpenCV: ${BUILD_OPENCV}")

if(NOT DEFINED OPENCV_GIT_TAG)
  SET(OPENCV_GIT_TAG 4af3ca4)
endif()
MESSAGE(STATUS "Using OPENCV_GIT_TAG: ${OPENCV_GIT_TAG}")
if(NOT DEFINED TRILINOS_GIT_TAG)
  SET(TRILINOS_GIT_TAG 094e4a2)
endif()
MESSAGE(STATUS "Using TRILINOS_GIT_TAG: ${TRILINOS_GIT_TAG}")

include(ExternalProject)

# Trilinos

SET(TRILINOS_CMAKE_ARGS -D CMAKE_INSTALL_PREFIX:PATH=${CMAKE_SOURCE_DIR}/trilinos -D CMAKE_CXX_COMPILER:FILEPATH=${CMAKE_CXX_COMPILER} -D CMAKE_C_COMPILER:FILEPATH=${CMAKE_C_COMPILER} -D CMAKE_Fortran_COMPILER:FILEPATH=${CMAKE_Fortran_COMPILER} -D CMAKE_BUILD_TYPE:STRING=RELEASE -D CMAKE_CXX_FLAGS:STRING=-g -Wall -D CMAKE_C_FLAGS:STRING=-g -Wall -D Trilinos_ENABLE_TriKota=OFF -D Trilinos_ENABLE_Triutils=OFF -D Trilinos_ENABLE_DEBUG:BOOL=OFF -D Trilinos_ENABLE_CHECKED_STL:BOOL=OFF -D Trilinos_ENABLE_EXPLICIT_INSTANTIATION:BOOL=ON -D Trilinos_WARNINGS_AS_ERRORS_FLAGS:STRING= -D Trilinos_ENABLE_ALL_PACKAGES:BOOL=OFF -D Trilinos_ENABLE_ALL_OPTIONAL_PACKAGES:BOOL=OFF -D BUILD_SHARED_LIBS:BOOL=OFF -D TPL_FIND_SHARED_LIBS=OFF -D Trilinos_LINK_SEARCH_START_STATIC=ON -D CMAKE_VERBOSE_MAKEFILE:BOOL=OFF -D Trilinos_ENABLE_CXX11:BOOL=ON -D TPL_ENABLE_MPI:BOOL=OFF -D Trilinos_ENABLE_OpenMP:BOOL=OFF -D Trilinos_ENABLE_Ifpack:BOOL=ON -D Trilinos_ENABLE_Ifpack2:BOOL=OFF -D TPL_ENABLE_CUDA:BOOL=OFF -D TPL_ENABLE_BinUtils=OFF -D TPL_ENABLE_BLAS:STRING=ON -D TPL_ENABLE_LAPACK:STRING=ON -D TPL_ENABLE_MKL:STRING=OFF -D TPL_ENABLE_HWLOC:STRING=OFF -D TPL_ENABLE_BLAS:BOOL=ON -D TPL_BLAS_LIBRARIES:FILEPATH=${BLAS} -D TPL_ENABLE_LAPACK:BOOL=ON -D TPL_LAPACK_LIBRARIES:FILEPATH=${LAPACK} -D Trilinos_ENABLE_SEACASExodus:BOOL=ON -D Trilinos_ENABLE_SEACASIoss:BOOL=OFF -D TPL_ENABLE_Netcdf:BOOL=ON -D Trilinos_ENABLE_Teuchos:BOOL=ON -D Trilinos_ENABLE_Belos:BOOL=ON -D Trilinos_ENABLE_Kokkos:BOOL=OFF -D Trilinos_ENABLE_Tpetra:BOOL=OFF -D Trilinos_ENABLE_Epetra:BOOL=ON  -D Netcdf_INCLUDE_DIRS:PATH=${CMAKE_CURRENT_SOURCE_DIR}/netcdf/include -D Netcdf_LIBRARY_DIRS:PATH=${CMAKE_CURRENT_SOURCE_DIR}/netcdf/lib)

# test for the installed include directory before downloading and building
if((NOT EXISTS "${CMAKE_SOURCE_DIR}/trilinos/include") AND ${BUILD_TRILINOS})
  ExternalProject_Add(trilinos
    GIT_REPOSITORY git@github.com:trilinos/Trilinos.git
    GIT_TAG ${TRILINOS_GIT_TAG}
    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/trilinos
    CMAKE_ARGS ${TRILINOS_CMAKE_ARGS}
  )
endif()

# OpenCV

SET(OPENCV_CMAKE_ARGS -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX:STRING=${CMAKE_SOURCE_DIR}/opencv -D BUILD_SHARED_LIBS=OFF -D CMAKE_CXX_COMPILER:FILEPATH=${CMAKE_CXX_COMPILER} -D CMAKE_C_COMPILER:FILEPATH=${CMAKE_C_COMPILER} -D CMAKE_Fortran_COMPILER:FILEPATH=${CMAKE_Fortran_COMPILER} -D BUILD_opencv_video=ON -D BUILD_opencv_static=ON -D BUILD_EXAMPLES=ON -D WITH_IPP=OFF -D OPENCV_SKIP_VISIBILITY_HIDDEN=True -D BUILD_TESTS=OFF -D BUILD_PERF_TESTS=OFF -D BUILD_EXAMPLES=OFF -D BUILD_opencv_apps=OFF)

# test for the installed include directory before downloading and building
if((NOT EXISTS "${CMAKE_SOURCE_DIR}/opencv/include") AND ${BUILD_OPENCV})
  ExternalProject_Add(opencv
    GIT_REPOSITORY git@github.com:opencv/opencv.git
    GIT_TAG ${OPENCV_GIT_TAG}
    PREFIX       ${CMAKE_CURRENT_BINARY_DIR}/opencv
    CMAKE_ARGS   ${OPENCV_CMAKE_ARGS}
  )
endif()

# electron

file(ARCHIVE_EXTRACT INPUT ${CMAKE_CURRENT_SOURCE_DIR}/electron/electron-v1.3.4-darwin-x64.zip DESTINATION ${CMAKE_SOURCE_DIR}/electron)
