cmake_minimum_required(VERSION 3.10)
project(pitch_detection VERSION 1.0 LANGUAGES CXX)

# Set C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# use this for dependency graph generation
#set_property(GLOBAL PROPERTY GRAPHVIZ_EXPORT_TARGETS TRUE)

set(CMAKE_CXX_FLAGS "-Wall -Wextra -ansi -pedantic -fext-numeric-literals -fopenmp")
set(CMAKE_CXX_FLAGS_DEBUG "-g")
set(CMAKE_CXX_FLAGS_RELEASE "-O3 -march=native -flto")

include_directories(include)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/test)

find_package(gflags QUIET)
find_package(libnyquist QUIET)
find_package(PkgConfig QUIET)
pkg_search_module(OPUS QUIET opus)
pkg_search_module(WAVPACK QUIET wavpack)

if(gflags_FOUND AND OPUS_FOUND AND WAVPACK_FOUND AND libnyquist_FOUND)
    file(GLOB WAV_ANALYZER_SOURCES "wav_analyzer/*.cpp")
    add_executable(wav_analyzer ${WAV_ANALYZER_SOURCES})
    target_link_libraries(wav_analyzer PRIVATE
        pitch_detection
        ${GFLAGS_LIBRARIES}
        ${OPUS_LIBRARIES}
        ${WAVPACK_LIBRARIES}
        libnyquist)

    target_include_directories(wav_analyzer PRIVATE
        ${GFLAGS_INCLUDE_DIRS}
        ${OPUS_INCLUDE_DIRS}
        ${WAVPACK_INCLUDE_DIRS}
        ${LIBNYQUIST_INCLUDE_DIRS})

    link_directories(
        ${GFLAGS_LIBRARY_DIRS}
        ${OPUS_LIBRARY_DIRS}
        ${WAVPACK_LIBRARY_DIRS}
        ${GFLAGS_LIBRARY_DIRS}
        ${OPUS_LIBRARY_DIRS}
        ${WAVPACK_LIBRARY_DIRS}
        ${LIBYNQUIST_LIBRARY_DIRS})
endif()

include(CTest)
find_package(GTest QUIET)
find_package(opus QUIET)
find_package(lib QUIET)
if(GTEST_FOUND)
    enable_testing()
    file(GLOB TEST_SOURCES "test/test*.cpp" "test/util.cpp")
    add_executable(pitch_tests ${TEST_SOURCES})
    target_link_libraries(pitch_tests PRIVATE
        pitch_detection
        GTest::GTest
        GTest::Main)
endif()

find_package(benchmark QUIET)
if(benchmark_FOUND)
    file(GLOB BENCH_SOURCES "test/bench.cpp" "test/util.cpp")
    add_executable(pitch_bench ${BENCH_SOURCES})
    target_link_libraries(pitch_bench PRIVATE
        pitch_detection
        benchmark::benchmark)
endif()

# Fetch external dependencies
include(FetchContent)
set(FETCHCONTENT_QUIET OFF)

# Fetch FFTS Library
FetchContent_Declare(
    ffts
    GIT_REPOSITORY https://github.com/anthonix/ffts.git
    GIT_SHALLOW TRUE
)
FetchContent_MakeAvailable(ffts)

# # Fetch Ensmallen Library
# FetchContent_Declare(
#     ensmallen
#     GIT_REPOSITORY https://github.com/mlpack/ensmallen.git
#     GIT_TAG 2.20.0  # Specify a specific version
#     GIT_SHALLOW TRUE
# )
# FetchContent_MakeAvailable(ensmallen)

# # Fetch Armadillo Library
# FetchContent_Declare(
#     armadillo
#     GIT_REPOSITORY https://gitlab.com/conradsnicta/armadillo-code
#     GIT_TAG 14.2.x  # Specify a specific version
#     GIT_SHALLOW TRUE
# )
# FetchContent_MakeAvailable(armadillo)

# # # Explicitly set Ensmallen include directory
# # set(ENSMALLEN_INCLUDE_DIR ${ensmallen_SOURCE_DIR}/include)

# Include directories
include_directories(
    ${CMAKE_SOURCE_DIR}/include
    ${ffts_SOURCE_DIR}/include
    # ${ENSMALLEN_INCLUDE_DIR}
    # ${mlpack_SOURCE_DIR}/src
)

# message(${ffts_SOURCE_DIR}/include)
# message(${ffts_SOURCE_DIR}/include)
# message(${ffts_SOURCE_DIR}/include)
# message(${ffts_SOURCE_DIR}/include)
# message(${ffts_SOURCE_DIR}/include)

# # Fetch mlpack Library
# set(CMAKE_SOURCE_DIR_BAK ${CMAKE_SOURCE_DIR})
# set(CMAKE_SOURCE_DIR ${mlpack_SOURCE_DIR}/src)
# FetchContent_Declare(
#     mlpack
#     GIT_REPOSITORY https://github.com/mlpack/mlpack.git
#     GIT_SHALLOW TRUE
# )
# FetchContent_MakeAvailable(mlpack)
# set(CMAKE_SOURCE_DIR ${CMAKE_SOURCE_DIR_BAK})

# Create static library
add_library(pitch_detection STATIC
    src/autocorrelation.cpp
    src/hmm.cpp
    src/mpm.cpp
    src/parabolic_interpolation.cpp
    src/yin.cpp
)

# Link libraries statically
target_link_libraries(pitch_detection 
    PRIVATE 
    ffts 
    mlpack
)

# Create executable
add_executable(wav_analyzer wav_analyzer/wav_analyzer.cpp)
target_link_libraries(wav_analyzer PRIVATE pitch_detection)

# Optional: Installation rules
install(TARGETS pitch_detection wav_analyzer
    LIBRARY DESTINATION lib
    RUNTIME DESTINATION bin
)

# Optional: Export headers
install(TARGETS pitch_detection LIBRARY DESTINATION lib)
install(FILES include/pitch_detection.h DESTINATION include)
