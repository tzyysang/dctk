cmake_minimum_required (VERSION 3.9)
MESSAGE(STATUS "CMAKE_ROOT: " ${CMAKE_ROOT})

# Project name
project(delay_calc_toolkit)

include(CTest)

# Turn on the verbose
set(CMAKE_VERBOSE_MAKEFILE ON)

# CXX target properties
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
set(USE_WERROR ON)
#set(CMAKE_BUILD_TYPE Debug)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -O2 -fopenmp")
message(STATUS "CMAKE_BUILD_TYPE: " ${CMAKE_BUILD_TYPE})
message(STATUS "CMAKE_CXX_COMPILER: " ${CMAKE_CXX_COMPILER})
message(STATUS "CMAKE_CXX_FLAGS: " ${CMAKE_CXX_FLAGS})
message(STATUS "CMAKE_EXE_LINKER_FLAGS: " ${CMAKE_EXE_LINKER_FLAGS})


# add the binary tree to the search path for include files
include_directories(${PROJECT_SOURCE_DIR})
include_directories(../yaml-cpp/include)
include_directories(src-liberty_parse-2.6)
include_directories(src)
include_directories(../Parser-SPEF/parser-spef)

# -----------------------------------------------------------------------------
# Example program 
# -----------------------------------------------------------------------------
message(STATUS "Building examples ...")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/build)

message(STATUS "EXAMPLE_CXX_FLAGS: " ${EXAMPLE_CXX_FLAGS})
message(STATUS "EXAMPLE_EXE_LINKER_FLAGS: " ${EXAMPLE_EXE_LINKER_FLAGS})

# YAML reader
link_directories(../yaml-cpp/build)

# Liberty parser
link_directories(src-liberty_parse-2.6)

# Data model
add_library(dctk src/Circuit.cpp src/liberty_reader.cpp src/CellLib.cpp src/Cell.cpp src/CellPin.cpp src/CellArc.cpp src/circuit_reader.cpp src/spice_lib_reader.cpp)

# contestant's delay calculation code/library
include_directories(src-dut)
link_directories(src-dut)
add_library(dut src-dut/delay_calculator.cpp src-dut/circuit_sim.cpp src-dut/current_table.cpp src-dut/delay_fall.cpp src-dut/delay_fall_lin.cpp src-dut/delay_rise.cpp src-dut/delay_rise_lin.cpp src-dut/os_service.cpp src-dut/receiver_table.cpp src-dut/score_delays.cpp src-dut/score_slews.cpp src-dut/slew_fall.cpp src-dut/slew_fall_lin.cpp src-dut/slew_rise.cpp src-dut/slew_rise_lin.cpp src-dut/waveform.cpp)

# benchmarking executable
add_executable(delay_calc_tool src/delay_calc_tool.cpp)

IF(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
  target_link_libraries(delay_calc_tool dctk yaml-cpp libparse dut)
ENDIF(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")

IF(${CMAKE_SYSTEM_NAME} MATCHES "Linux")
  target_link_libraries(delay_calc_tool dctk yaml-cpp libparse dut stdc++fs)
ENDIF(${CMAKE_SYSTEM_NAME} MATCHES "Linux")

# Random Net Generator
add_executable(gen_random_nets src/gen_random_nets.cpp)
target_link_libraries(gen_random_nets dctk libparse yaml-cpp)

