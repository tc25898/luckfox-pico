#1.cmake version
cmake_minimum_required(VERSION 3.16...4.2)

#2.project name
project(video_algorithm)

if (MOVE_DETECT_ENABLE)
add_subdirectory(move_detect)
endif()

if (OCCLUSION_DETECT_ENABLE)
add_subdirectory(occlusion_detect)
endif()