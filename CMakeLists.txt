cmake_minimum_required(VERSION 3.15)

project(poly2tri LANGUAGES CXX)

include(GNUInstallDirs)

include(FetchContent)
FetchContent_Declare(
  gsl
  GIT_REPOSITORY https://github.com/microsoft/GSL.git
  GIT_TAG 303d964a241cb2f2a058fe0baa14013eb9afa116)
set(GSL_INSTALL ON)
FetchContent_MakeAvailable(gsl)

option(P2T_BUILD_TESTS "Build tests" OFF)
option(P2T_BUILD_TESTBED "Build the testbed application" OFF)

file(GLOB SOURCES poly2tri/common/*.cc poly2tri/sweep/*.cc)
file(GLOB HEADERS poly2tri/*.h poly2tri/common/*.h poly2tri/sweep/*.h)

add_library(poly2tri)
target_sources(
  poly2tri
  PRIVATE poly2tri/common/shapes.cc poly2tri/sweep/advancing_front.cc
          poly2tri/sweep/sweep_context.cc poly2tri/sweep/sweep.cc
          poly2tri/sweep/cdt.cc)

target_include_directories(
  poly2tri PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
                  $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)
target_link_libraries(poly2tri PUBLIC GSL)
target_compile_features(poly2tri PUBLIC cxx_std_17)

set_target_properties(poly2tri PROPERTIES DEBUG_POSTFIX "d")

get_target_property(poly2tri_target_type poly2tri TYPE)
if(poly2tri_target_type STREQUAL SHARED_LIBRARY)
  target_compile_definitions(poly2tri PRIVATE P2T_SHARED_EXPORTS)
else()
  target_compile_definitions(poly2tri PUBLIC P2T_STATIC_EXPORTS)
endif()

if(P2T_BUILD_TESTS)
  enable_testing()
  add_subdirectory(unittest)
endif()

if(P2T_BUILD_TESTBED)
  add_subdirectory(testbed)
endif()

install(
  TARGETS poly2tri
  EXPORT poly2tri-config
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})

install(DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/poly2tri
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

install(
  EXPORT poly2tri-config
  NAMESPACE poly2tri::
  FILE poly2tri-targets.cmake
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/poly2tri)

configure_file(poly2tri-config.cmake.in poly2tri-config.cmake @ONLY)
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/poly2tri-config.cmake"
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/poly2tri)
