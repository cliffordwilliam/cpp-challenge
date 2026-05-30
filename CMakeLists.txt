cmake_minimum_required(VERSION 3.23)
project(sprout-lands LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

add_subdirectory(vendored/SDL EXCLUDE_FROM_ALL)

add_executable(main)
target_sources(main PRIVATE main.cpp)

target_link_libraries(main PRIVATE SDL3::SDL3)
