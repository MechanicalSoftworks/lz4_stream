cmake_minimum_required(VERSION 3.1)

project(lz4_stream LANGUAGES CXX VERSION "1.0.0")

include(ExternalProject)

option(LZ4_STREAM_UseBundledLz4 "Use bundled static LZ4 library" OFF)
option(LZ4_STREAM_BuildTests "Build unittests" ON)
option(LZ4_STREAM_BuildExamples "Build examples" ON)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

set(LZ4_STREAM_LIBRARY_NAME ${PROJECT_NAME})
set(LZ4_STREAM_LIBRARY_INCLUDE_DIR "${PROJECT_SOURCE_DIR}/include/")

# TODO: Probably shouldn't be set like this
set(CMAKE_CXX_STANDARD 11)
if(MSVC)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W3 /WX /EHsc")
else()
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -Werror")
endif()

# TODO: Probably shouldn't be set like this
set(CMAKE_CXX_FLAGS_COVERAGE
  "${GCC_DEBUG_FLAGS} -fprofile-arcs -ftest-coverage"
  CACHE STRING "Flags used by the C++ compiler during coverage builds."
  FORCE)
set(CMAKE_EXE_LINKER_FLAGS_COVERAGE
  "${CMAKE_EXE_LINKER_FLAGS} --coverage"
  CACHE STRING "Flags used for linking binaries during coverage builds."
  FORCE)
mark_as_advanced(
  CMAKE_CXX_FLAGS_COVERAGE
  CMAKE_EXE_LINKER_FLAGS_COVERAGE)

# Create lz4_stream library
add_library(${LZ4_STREAM_LIBRARY_NAME} INTERFACE)
target_include_directories(${LZ4_STREAM_LIBRARY_NAME}
  INTERFACE
    $<BUILD_INTERFACE:${LZ4_STREAM_LIBRARY_INCLUDE_DIR}>
    $<INSTALL_INTERFACE:$<INSTALL_PREFIX>/${CMAKE_INSTALL_INCLUDEDIR}>
)

file(GLOB_RECURSE HEADER_LIST CONFIGURE_DEPENDS "${LZ4_STREAM_LIBRARY_INCLUDE_DIR}*")
set_target_properties(${LZ4_STREAM_LIBRARY_NAME} PROPERTIES
    PUBLIC_HEADER "${HEADER_LIST}"
)

if(LZ4_STREAM_BuildTests)
  enable_testing()
  add_subdirectory("thirdparty/catch2")
  add_subdirectory("test")
endif()

if(LZ4_STREAM_BuildExamples)
  add_executable(lz4_compress lz4_compress.cpp)
  target_link_libraries(lz4_compress ${LZ4_STREAM_LIBRARY_NAME})

  add_executable(lz4_decompress lz4_decompress.cpp)
  target_link_libraries(lz4_decompress ${LZ4_STREAM_LIBRARY_NAME})
endif()

install(TARGETS ${LZ4_STREAM_LIBRARY_NAME}
  EXPORT ${LZ4_STREAM_LIBRARY_NAME}Targets
)

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    ${LZ4_STREAM_LIBRARY_NAME}ConfigVersion.cmake
    VERSION ${PACKAGE_VERSION}
    COMPATIBILITY ExactVersion
)

install(EXPORT ${LZ4_STREAM_LIBRARY_NAME}Targets
	FILE ${LZ4_STREAM_LIBRARY_NAME}Targets.cmake
	DESTINATION ${CMAKE_INSTALL_CMAKEDIR}/${LZ4_STREAM_LIBRARY_NAME}
)

configure_file(${PROJECT_SOURCE_DIR}/cmake/Config.cmake.in ${CMAKE_CURRENT_BINARY_DIR}/${LZ4_STREAM_LIBRARY_NAME}Config.cmake @ONLY)
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/${LZ4_STREAM_LIBRARY_NAME}Config.cmake"
	"${CMAKE_CURRENT_BINARY_DIR}/${LZ4_STREAM_LIBRARY_NAME}ConfigVersion.cmake"
	DESTINATION ${CMAKE_INSTALL_CMAKEDIR}/${LZ4_STREAM_LIBRARY_NAME}
)