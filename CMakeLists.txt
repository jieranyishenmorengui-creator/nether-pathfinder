cmake_minimum_required(VERSION 3.16)
project(nether_pathfinder)

option(PATHFINDER_WIN7_COMPAT "Build with Windows 7 API target (0x0601)" ON)

find_package(JNI COMPONENTS JNI_INCLUDE_DIRS)
include_directories(${JNI_INCLUDE_DIRS})

find_package(PkgConfig) # not required
if(PkgConfig_FOUND)
    pkg_check_modules(GBENCHMARK IMPORTED_TARGET benchmark) # not required
endif()
set(CMAKE_C_COMPILER_TARGET ${PATHFINDER_TARGET})
set(CMAKE_CROSSCOMPILING True)
set(WITH_SANITIZER False)
set(ZLIBNG_ENABLE_TESTS False)
set(ZLIB_ENABLE_TESTS False)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
add_subdirectory(zlib-ng)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall")
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE}")
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -O0")
message("${CMAKE_CXX_COMPILER_ID} ${CMAKE_CXX_COMPILER_VERSION}")

file(GLOB_RECURSE SRC src/*)
list(REMOVE_ITEM SRC "${CMAKE_CURRENT_SOURCE_DIR}/src/benchmark_main.cpp" "${CMAKE_CURRENT_SOURCE_DIR}/src/main.cpp" "${CMAKE_CURRENT_SOURCE_DIR}/src/baritone_pather.cpp")

if (TESTING)
    add_executable(nether_pathfinder ${SRC} src/main.cpp)
    if(GBENCHMARK_FOUND)
        #add_executable(nether_pathfinder_benchmark ${SRC} src/benchmark_main.cpp)
        #target_link_libraries(nether_pathfinder_benchmark PkgConfig::GBENCHMARK)
        #target_include_directories(nether_pathfinder_benchmark PRIVATE ${CMAKE_CURRENT_BINARY_DIR}/zlib-ng)
    endif()
else()
    #add_executable(baritone_pather ${SRC} src/baritone_pather.cpp)
    #target_include_directories(baritone_pather PRIVATE ${CMAKE_CURRENT_BINARY_DIR}/zlib-ng)
    #target_link_libraries(baritone_pather PRIVATE zlibstatic)

    add_library(nether_pathfinder SHARED ${SRC})
endif()
target_include_directories(nether_pathfinder PRIVATE ${CMAKE_CURRENT_BINARY_DIR}/zlib-ng)
target_link_libraries(nether_pathfinder PRIVATE zlibstatic)
target_compile_features(nether_pathfinder PRIVATE cxx_std_20)

if (WIN32 AND PATHFINDER_WIN7_COMPAT)
    target_compile_definitions(nether_pathfinder PRIVATE WINVER=0x0601 _WIN32_WINNT=0x0601)
endif()

if (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    message(WARNING "gcc is not recommended")
elseif (CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
    if (CMAKE_CXX_COMPILER_VERSION VERSION_LESS 13.0)
        message(FATAL_ERROR "requires clang at least clang 13")
    endif()
endif()

#target_link_options(nether_pathfinder PRIVATE "-static-libgcc" "-static-libstdc++")
if (USE_ABSEIL)
    add_subdirectory(abseil-cpp)
    target_link_libraries(nether_pathfinder absl::flat_hash_map)
endif()
