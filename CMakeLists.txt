cmake_minimum_required(VERSION 2.8.7)

# faiss project
project(faiss C CXX)

option(BUILD_TUTORIAL "Build tutorials" ON)
option(BUILD_TEST "Build tests" OFF)
option(BUILD_WITH_GPU "Build faiss with gpu (cuda) support" OFF)
option(WITH_MKL "Build with MKL if ON (OpenBLAS if OFF)" OFF)

list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake/Modules)

# Mac for gcc & g++
set(CMAKE_C_COMPILER "/usr/local/Cellar/llvm/7.0.1/bin/clang")
set(CMAKE_CXX_COMPILER "/usr/local/Cellar/llvm/7.0.1/bin/clang++")
# Mac for OpenMP
set(OPENMP_LIBRARIES "/usr/local/Cellar/llvm/7.0.1/lib")
set(OPENMP_INCLUDES "/usr/local/Cellar/llvm/7.0.1/include")
# OpenMP
option(USE_OpenMP "Use OpenMP to enamble <omp.h>" ON)
# Find OpenMP
if(APPLE AND USE_OpenMP)
    if(CMAKE_C_COMPILER_ID MATCHES "Clang")
        set(OpenMP_C "${CMAKE_C_COMPILER}")
        set(OpenMP_C_FLAGS "-fopenmp=libomp -Wno-unused-command-line-argument")
        set(OpenMP_C_LIB_NAMES "libomp" "libgomp" "libiomp5")
        set(OpenMP_libomp_LIBRARY ${OpenMP_C_LIB_NAMES})
        set(OpenMP_libgomp_LIBRARY ${OpenMP_C_LIB_NAMES})
        set(OpenMP_libiomp5_LIBRARY ${OpenMP_C_LIB_NAMES})
    endif()
    if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
        set(OpenMP_CXX "${CMAKE_CXX_COMPILER}")
        set(OpenMP_CXX_FLAGS "-fopenmp=libomp -Wno-unused-command-line-argument")
        set(OpenMP_CXX_LIB_NAMES "libomp" "libgomp" "libiomp5")
        set(OpenMP_libomp_LIBRARY ${OpenMP_CXX_LIB_NAMES})
        set(OpenMP_libgomp_LIBRARY ${OpenMP_CXX_LIB_NAMES})
        set(OpenMP_libiomp5_LIBRARY ${OpenMP_CXX_LIB_NAMES})
    endif()
endif()

if(USE_OpenMP)
    find_package(OpenMP REQUIRED)
endif(USE_OpenMP)

if (OPENMP_FOUND)
    include_directories("${OPENMP_INCLUDES}")
    message(STATUS "Found MAC_OpenMP include: ${OPENMP_INCLUDES}")
    link_directories("${OPENMP_LIBRARIES}")
    message(STATUS "Found MAC_OpenMP libraries: ${OPENMP_LIBRARIES}")
    set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
    set (CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_EXE_LINKER_FLAGS}")
endif(OPENMP_FOUND)

# BLAS (MKL os OpenBLAS)
if(WITH_MKL)
    find_package(MKL REQUIRED)
    include_directories(${MKL_INCLUDE_DIRS})
    set(BLAS_LIB ${MKL_LIBRARIES})
else()
    find_package(OpenBLAS REQUIRED)
    include_directories(${OpenBLAS_INCLUDE_DIR})
    set(BLAS_LIB ${OpenBLAS_LIB})
endif()

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -fPIC -m64 -Wall -g -O3  -msse4 -mpopcnt -fopenmp -Wno-sign-compare")
add_definitions(-DFINTEGER=int)

# specify output bin_path and lib_path
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

# specify header and cpp files
file(GLOB faiss_cpu_headers ${CMAKE_CURRENT_SOURCE_DIR}/*.h)
file(GLOB faiss_cpu_cpp ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp)

set(faiss_lib faiss)
add_library(${faiss_lib} STATIC ${faiss_cpu_headers} ${faiss_cpu_cpp})
target_link_libraries(${faiss_lib} ${OpenMP_CXX_FLAGS} ${BLAS_LIB})

# build gpu lib
#if(BUILD_WITH_GPU)
    #include(cmake/Cuda.cmake)
    #add_subdirectory(gpu)
#endif(BUILD_WITH_GPU)

# build tutorial examples
if(BUILD_TUTORIAL)
    add_subdirectory(tutorial)
endif(BUILD_TUTORIAL)

# build tests
#if(BUILD_TEST)
    #add_subdirectory(tests)
#endif(BUILD_TEST)

# Install libraries
install(TARGETS ${faiss_lib}
        ARCHIVE DESTINATION lib
        )
install(FILES ${faiss_cpu_headers} DESTINATION include/faiss)
