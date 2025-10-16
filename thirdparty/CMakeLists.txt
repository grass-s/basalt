cmake_minimum_required(VERSION 3.10...3.18)


add_subdirectory(ros EXCLUDE_FROM_ALL)


set(BUILD_SHARED_LIBS OFF CACHE BOOL "Enable BUILD_SHARED_LIBS")
set(BUILD_TESTS OFF CACHE BOOL "Enable BUILD_TESTS")
set(BUILD_TOOLS OFF CACHE BOOL "Enable BUILD_TOOLS")
set(CMAKE_EXPORT_NO_PACKAGE_REGISTRY ON)


# Hack to disable march=native in opengv
macro(add_definitions)
  if(NOT ${ARGV0} STREQUAL "-march=native")
    _add_definitions(${ARGN})
  endif()
endmacro()
add_subdirectory(opengv EXCLUDE_FROM_ALL)

# Hack to disable CPack in Pangolin.
macro(include)
  if(NOT ${ARGV0} STREQUAL "CPack")
    _include(${ARGN})
  endif()
endmacro()

