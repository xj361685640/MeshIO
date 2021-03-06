CMAKE_MINIMUM_REQUIRED(VERSION 3.0.2)

PROJECT(MeshIO)

SET(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_SOURCE_DIR}/CMakeModules")

OPTION(BUILD_DOCUMENTATION "Build Documentation" OFF)
OPTION(BUILD_EXAMPLES "Build examples" OFF)
OPTION(BUILD_TESTS "Build Tests" ON)
OPTION(BUILD_COVERAGE "Create test coverage report" OFF)
OPTION(USE_SYSTEM_GTEST "Use GTEST from system libraries" OFF)
OPTION(USE_RELATIVE_TEST_DIR "Use relative paths for the test data directory(For continious integration(CI) purposes only)" OFF)

# Set a default build type if none was specified
IF(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    SET(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build." FORCE)
    # Set the possible values of build type for cmake-gui
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release"
      "MinSizeRel" "RelWithDebInfo")
ENDIF()

### Marking BUILD_COVERAGE as advanced as it won't be
### used by regular developers
MARK_AS_ADVANCED(BUILD_COVERAGE)

IF(UNIX)
    IF("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
        SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
    ELSEIF("${CMAKE_CXX_COMPILER_VERSION}" VERSION_LESS 4.7)
        SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
    ELSE()
        SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
    ENDIF()
ENDIF()

# Generate documentation
IF(BUILD_DOCUMENTATION)
    ADD_SUBDIRECTORY(docs)
ENDIF(BUILD_DOCUMENTATION)

IF(BUILD_EXAMPLES)
    ADD_SUBDIRECTORY(examples)
ENDIF()

IF(BUILD_TESTS)
    ENABLE_TESTING()
    ADD_SUBDIRECTORY(test)
ENDIF()

### Following commented out changes work only on CMAKE version >=3.0
ADD_LIBRARY(MeshIO INTERFACE)
TARGET_INCLUDE_DIRECTORIES(MeshIO INTERFACE
           $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
           $<INSTALL_INTERFACE:include/meshio>
           )

INSTALL(TARGETS MeshIO EXPORT meshioExport)
INSTALL(EXPORT meshioExport NAMESPACE Upstream::
       DESTINATION share/MeshIO/cmake
       )

INSTALL(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/include"
        DESTINATION .
        COMPONENT Headers
        FILES_MATCHING
        PATTERN "*.hpp"
        )
