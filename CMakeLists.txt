cmake_minimum_required(VERSION 3.14)

project(
    "rva"
    VERSION 0.9.5
    DESCRIPTION "A Simple Library for Recursive Variant Types"
    HOMEPAGE_URL "https://github.com/codeinred/recursive-variant/"
    LANGUAGES CXX
)

add_library(rva INTERFACE)
add_library(rva::rva ALIAS rva)

# For ${CMAKE_INSTALL_<DIR>} variables that are standarized
include(GNUInstallDirs)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake")
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

target_compile_features(
    rva
    INTERFACE
    cxx_std_20
)

target_include_directories(
    rva
    INTERFACE
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

# Installation
# See: https://dominikberner.ch/cmake-interface-lib/

install(
    TARGETS rva
    EXPORT ${PROJECT_NAME}_Targets
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    "${PROJECT_NAME}ConfigVersion.cmake"
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion
)

configure_package_config_file(
    "${PROJECT_SOURCE_DIR}/cmake/config.cmake.in"
    "${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
    INSTALL_DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/${PROJECT_NAME}/cmake
)

install(
    EXPORT ${PROJECT_NAME}_Targets
    FILE ${PROJECT_NAME}Targets.cmake
    NAMESPACE rva::
    DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/${PROJECT_NAME}/cmake
)

install(FILES "${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
              "${PROJECT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
        DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/${PROJECT_NAME}/cmake
)

install(DIRECTORY ${PROJECT_SOURCE_DIR}/include/rva
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)


include(helper)

option(
    BUILD_TESTING
    "Build testing for RVA"
    ON)

if(PROJECT_IS_TOP_LEVEL AND BUILD_TESTING)
    include(CTest)

    find_or_fetch(
        Catch2
        https://github.com/catchorg/catch2.git
        devel
        3.0.0)
    FetchContent_MakeAvailable(${remote_dependencies})

    add_executable(
        test_rva
        test/test_rva.cpp)
    target_link_libraries(
        test_rva
        rva::rva
        Catch2::Catch2WithMain)

    list(APPEND CMAKE_MODULE_PATH ${CMAKE_INSTALL_PREFIX}/lib/cmake/Catch2)
    list(APPEND CMAKE_MODULE_PATH ${catch2_SOURCE_DIR}/extras)
    include(CTest)
    include(Catch)
    catch_discover_tests(test_rva)
endif()


