#
# CMakeLists.txt
# cpds
#
# Copyright (c) 2016 Hannes Friederich.
#

cmake_minimum_required(VERSION 3.1)
project("cpds" CXX)

set (CPDS_MAJOR_VERSION 0)
set (CPDS_MINOR_VERSION 1)
set (CPDS_PATCH_VERSION 0)

set (CPDS_VERSION
  ${CPDS_MAJOR_VERSION}.${CPDS_MINOR_VERSION}.${CPDS_PATCH_VERSION})

set(CMAKE_CXX_STANDARD 11)

# location of custom CMake modules
list (APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake_modules)

option(WITH_JSON          "Enable JSON support" ON)
option(WITH_YAML          "Enable YAML support (requires yaml-cpp)" OFF)
option(BUILD_SHARED_LIBS  "Build Shared Libraries" OFF)
option(BUILD_TESTS        "Builds the test suite (requires GTest)" OFF)

if(BUILD_TESTS)
  set(WITH_YAML ON)
endif()

if(WITH_YAML)
  find_package(YAMLCPP REQUIRED)
endif()

if(BUILD_TESTS)
  find_package(GTest REQUIRED)
  find_package(Threads REQUIRED)
endif()

include_directories(./include)

set(CORE_FILES
  include/cpds/exception.hpp
  include/cpds/typedefs.hpp
  include/cpds/node.hpp
  include/cpds/validator.hpp
  include/cpds/parsemark.hpp
  include/cpds/parseinfo.hpp
  src/exception.cpp
  src/node.cpp
  src/validator.cpp
  src/parseinfo.cpp
)

set(JSON_FILES
  include/cpds/json.hpp
  src/json.cpp
)

set(YAML_FILES
  include/cpds/yaml.hpp
  src/yaml.cpp
)

set(TEST_FILES
  test/test_exception.cpp
  test/test_node.cpp
  test/test_json.cpp
  test/test_yaml.cpp
  test/test_validator.cpp
  test/test_parseinfo.cpp
)

set(BUILD_FILES ${CORE_FILES})
if(WITH_JSON)
  set(BUILD_FILES
    ${BUILD_FILES}
    ${JSON_FILES}
  )
endif()
if(WITH_YAML)
  set(BUILD_FILES
    ${BUILD_FILES}
    ${YAML_FILES}
  )
endif()

set_source_files_properties(
  ${CORE_FILES} ${JSON_FILES} ${YAML_FILES}
  PROPERTIES COMPILE_FLAGS
  "-Werror -Wall -Wextra -pedantic -fno-common"
)

add_library(cpds ${BUILD_FILES})
set_target_properties(cpds PROPERTIES VERSION ${CPDS_MAJOR_VERSION})
set_target_properties(cpds PROPERTIES SOVERSION ${CPDS_VERSION})
if(WITH_YAML)
  target_link_libraries(cpds ${YAMLCPP_LIBRARIES})
endif()
install(TARGETS cpds DESTINATION lib)

if(BUILD_TESTS)
  add_executable(cpds-test ${TEST_FILES})
  target_include_directories(cpds-test PUBLIC ${GTEST_INCLUDE_DIRS})
  target_link_libraries(cpds-test cpds)
  target_link_libraries(cpds-test ${GTEST_BOTH_LIBRARIES})
  target_link_libraries(cpds-test ${CMAKE_THREAD_LIBS_INIT})
endif()

install(DIRECTORY include/cpds/
        DESTINATION include/cpds
        FILES_MATCHING PATTERN "*.hpp"
)

message(STATUS "-----------------------------------------")
message(STATUS "CMAKE_BUILD_TYPE  = ${CMAKE_BUILD_TYPE}")
message(STATUS "BUILD_SHARED_LIBS = ${BUILD_SHARED_LIBS}")
message(STATUS "WITH_JSON         = ${WITH_JSON}")
message(STATUS "WITH_YAML         = ${WITH_YAML}")
message(STATUS "BUILD_TESTS       = ${BUILD_TESTS}")
message(STATUS "-----------------------------------------")
