cmake_minimum_required(VERSION 3.5.0)
project(lct VERSION 0.1.0 LANGUAGES C CXX)

include(CTest)
enable_testing()

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib")
set(CMAKE_PDB_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib")

add_executable(lct main.cpp)
add_executable(rttest raytracing.cpp)
add_executable(wt windowtest.cpp)
add_executable(st sampletest.cpp)
add_executable(sc seamcarving.cpp)

#
# BEGIN LUISA COMPUTE
#
add_library(luisa-render-include INTERFACE)
target_include_directories(luisa-render-include INTERFACE ${CMAKE_CURRENT_SOURCE_DIR})
target_compile_features(luisa-render-include INTERFACE c_std_11 cxx_std_20)

if (CMAKE_CXX_COMPILER_ID MATCHES "MSVC")
    target_compile_options(luisa-render-include INTERFACE
            /wd4068 # Maxwell loves custom pragmas
            /wd5105 # winbase.h: macro expansion producing 'defined' has undefined behavior
            /Zc:preprocessor)
endif ()

set(LUISA_COMPUTE_ENABLE_CPU OFF CACHE BOOL "" FORCE)
set(LUISA_COMPUTE_BUILD_TESTS OFF CACHE BOOL "" FORCE)
set(LUISA_COMPUTE_ENABLE_LTO OFF CACHE BOOL "" FORCE)
set(LUISA_COMPUTE_ENABLE_PYTHON OFF CACHE BOOL "" FORCE)
set(LUISA_COMPUTE_ENABLE_UNITY_BUILD OFF CACHE BOOL "" FORCE)
add_subdirectory(compute)
target_include_directories(luisa-render-include INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/compute/include/luisa)
target_link_libraries(lct PUBLIC luisa::compute luisa-render-include)
target_link_libraries(rttest PUBLIC luisa::compute luisa-render-include)
target_link_libraries(wt PUBLIC luisa::compute luisa-render-include)
target_link_libraries(st PUBLIC luisa::compute luisa-render-include)
target_link_libraries(sc PUBLIC luisa::compute luisa-render-include)
#
# END LUISA COMPUTE
#

#
# BEGIN LODEPNG
#
add_library(lct-lib STATIC ext/lodepng/lodepng.cpp)
target_include_directories(lct-lib PUBLIC ext/lodepng)
target_link_libraries(lct PUBLIC lct-lib)
target_link_libraries(rttest PUBLIC lct-lib)
target_link_libraries(st PUBLIC lct-lib)
target_link_libraries(sc PUBLIC lct-lib)
#
# END LODEPNG
#

#
# BEGIN TINYOBJLOADER
#
add_subdirectory(ext/tinyobjloader)
target_link_libraries(rttest PUBLIC tinyobjloader)
#
# END TINYOBJLOADER
#

set(CPACK_PROJECT_NAME ${PROJECT_NAME})
set(CPACK_PROJECT_VERSION ${PROJECT_VERSION})
include(CPack)
