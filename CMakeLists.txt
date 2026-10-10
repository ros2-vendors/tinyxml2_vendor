cmake_minimum_required(VERSION 3.7)

project(tinyxml2_vendor)

find_package(ament_cmake REQUIRED)

option(FORCE_BUILD_VENDOR_PKG
  "Build tinyxml2 from source, even if a system-installed package is available"
  OFF)

if(NOT FORCE_BUILD_VENDOR_PKG)
  # Match Ubuntu 26.04 (lyrical target): libtinyxml2-dev 11.0.0
  find_package(tinyxml2 11 QUIET)
endif()

macro(build_tinyxml2)
  set(extra_cmake_args)
  if(DEFINED CMAKE_BUILD_TYPE)
    list(APPEND extra_cmake_args -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE})
  endif()
  if(DEFINED BUILD_SHARED_LIBS)
    list(APPEND extra_cmake_args -DBUILD_SHARED_LIBS=${BUILD_SHARED_LIBS})
  else()
    list(APPEND extra_cmake_args -DBUILD_SHARED_LIBS=ON)
  endif()
  if(DEFINED CMAKE_TOOLCHAIN_FILE)
    list(APPEND extra_cmake_args "-DCMAKE_TOOLCHAIN_FILE=${CMAKE_TOOLCHAIN_FILE}")
  endif()
  list(APPEND extra_cmake_args "-DCMAKE_CXX_FLAGS=${CMAKE_CXX_FLAGS}")
  list(APPEND extra_cmake_args -Dtinyxml2_BUILD_TESTING=OFF)

  include(ExternalProject)

  externalproject_add(tinyxml2_external
    GIT_REPOSITORY https://github.com/leethomason/tinyxml2.git
    GIT_TAG 11.0.0
    GIT_CONFIG advice.detachedHead=false
    UPDATE_COMMAND ""
    TIMEOUT 600
    CMAKE_ARGS
      -DCMAKE_INSTALL_PREFIX=${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}_install
      ${extra_cmake_args}
  )

  install(
    DIRECTORY
      ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}_install/
    DESTINATION
      ${CMAKE_INSTALL_PREFIX}
    USE_SOURCE_PERMISSIONS
  )
endmacro()

if(NOT tinyxml2_FOUND)
  message(STATUS "Did not find tinyxml2 11 installation, building from source")
  build_tinyxml2()
endif()

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package(
  CONFIG_EXTRAS "tinyxml2_vendor-extras.cmake"
)
