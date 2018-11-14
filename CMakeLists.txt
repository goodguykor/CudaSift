cmake_minimum_required(VERSION 2.6)

project(cudaSift)
set(cudaSift_VERSION_MAJOR 2)
set(cudaSift_VERSION_MINOR 0)
set(cudaSift_VERSION_PATCH 0)

set(CPACK_PACKAGE_VERSION_MAJOR "${cudaSift_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${cudaSift_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${cudaSift_VERSION_PATCH}")
set(CPACK_GENERATOR "ZIP")
include(CPack)

find_package(OpenCV REQUIRED)
find_package(CUDA)
if (NOT CUDA_FOUND)
  message(STATUS "CUDA not found. Project will not be built.")
endif(NOT CUDA_FOUND)

if (WIN32)
  set(EXTRA_CXX_FLAGS "/D_CRT_SECURE_NO_WARNINGS ")
  list(APPEND CUDA_NVCC_FLAGS "-arch=sm_35;--compiler-options;-O2") 
endif()
if (UNIX)
  if (APPLE)
    set(EXTRA_CXX_FLAGS "-msse2")
    list(APPEND CUDA_NVCC_FLAGS "-arch=sm_35;--compiler-options;-O2") 
  else()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O2")
    list(APPEND CUDA_NVCC_FLAGS "-lineinfo;--compiler-options;-O2") 
  endif()
endif()

set(cuda_sources
  # dynamic.cu
  cudaImage.cu  
  cudaImage.h  
  cudaSiftH.cu 
  cudaSiftH.h  
  matching.cu  
  cudaSiftD.h  
  cudaSift.h  
  cudautils.h
)  

set(sources
  geomFuncs.cpp  
  mainSift.cpp
)

include_directories(
  ${CMAKE_CURRENT_SOURCE_DIR} 
)

SET(CUDA_SEPARABLE_COMPILATION ON)

cuda_add_library(cudasift STATIC ${cuda_sources} ${sources} OPTIONS -arch=sm_35 -Xcompiler -fPIC)

target_compile_options(cudasift PRIVATE -fPIC)

set_target_properties(cudasift PROPERTIES
  COMPILE_FLAGS "${EXTRA_CXX_FLAGS}"			   
)

target_include_directories(cudasift INTERFACE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
)

target_link_libraries(cudasift
  /usr/local/cuda/lib64/libcudadevrt.a ${OpenCV_LIBS} 
)


install(FILES 
  ${cuda_sources} 
  ${sources}
  cudaSiftD.cu
  DESTINATION include
)

install(TARGETS cudasift EXPORT CudaSift
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
)
install(EXPORT CudaSift DESTINATION share/CudaSift/cmake)
