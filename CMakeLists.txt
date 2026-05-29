cmake_minimum_required(VERSION 3.23)
project("Sprout Lands" LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

add_executable(main)
target_sources(main PRIVATE main.cpp)