#1.cmake version
cmake_minimum_required(VERSION 3.16...4.2)

project(common_algorithm LANGUAGES C CXX)

option(COMPILES_EXAMPLES "Enable compiles examples" OFF)
add_subdirectory(audio)
add_subdirectory(video)
add_subdirectory(misc)
