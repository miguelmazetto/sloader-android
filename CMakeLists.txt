cmake_minimum_required(VERSION 3.16)
project(sloader)

enable_testing()
set(CMAKE_CXX_STANDARD 20)
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE "RelWithDebInfo")
endif()

set(CUSTOM_LIBC_PATH CACHE PATH "")

set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

include(third-party)
get_third_party(glog)
set(WITH_GFLAGS
    OFF
    CACHE INTERNAL "" FORCE)
set(WITH_UNWIND OFF)
set(BUILD_SHARED_LIBS OFF)
add_subdirectory(${CMAKE_CURRENT_BINARY_DIR}/glog
                 ${CMAKE_CURRENT_BINARY_DIR}/glog EXCLUDE_FROM_ALL)
include_directories(${CMAKE_CURRENT_BINARY_DIR}/glog
                    ${CMAKE_CURRENT_BINARY_DIR}/glog/src)
add_definitions(-DC10_USE_GLOG=1)

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/raw_write)

add_library(tls_secure tls_secure.cc)

add_executable(sloader sloader.cc dyn_loader.cc libc_mapping.cc utils.cc)
target_link_libraries(sloader glog tls_secure)
target_compile_options(sloader PUBLIC -Wall -Werror)
# TODO static link fail to secure TLS space.
# target_link_options(sloader PUBLIC -static -T
#                     ${CMAKE_CURRENT_SOURCE_DIR}/sloader.map)
target_link_options(sloader PUBLIC -T
                    ${CMAKE_CURRENT_SOURCE_DIR}/sloader.map)

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/CTestCustom.cmake
               ${CMAKE_CURRENT_BINARY_DIR})

add_subdirectory(tests)
