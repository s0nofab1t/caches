cmake_minimum_required(VERSION 3.15)

project(caches
        VERSION 0.1.1)

macro(add_coverage_flags target)
    if (CACHES_BUILD_TEST AND CACHES_ENABLE_COVERAGE)
        target_compile_options(${target} PRIVATE --coverage)

        if (NOT MSVC)
            target_link_options(${target} PRIVATE --coverage)
        endif ()
    endif ()
endmacro()

if (MSVC)
    add_definitions(/MP)
endif ()

set(CMAKE_CXX_STANDARD 11)

option(CACHES_INSTALL_LIBRARY "Install caches library" ON)
option(CACHES_BUILD_TEST "Build tests for the project" ON)
option(CACHES_ENABLE_COVERAGE "Build test executables with coverage support" OFF)

find_package(Doxygen)
if (DOXYGEN_FOUND)
    configure_file(${PROJECT_SOURCE_DIR}/cmake/Doxygen.in ${PROJECT_SOURCE_DIR}/docs/doxygen/Doxyfile)
    configure_file(${PROJECT_SOURCE_DIR}/cmake/index.md.in ${PROJECT_SOURCE_DIR}/docs/doxygen/index.md)

    add_custom_target(docs doxygen Doxyfile
        WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}/docs/doxygen
        COMMENT "Generate library Doxygen documentation")
endif ()

add_library(caches INTERFACE)

target_include_directories(caches INTERFACE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)

if (CACHES_BUILD_TEST)
    enable_testing()
    add_custom_target(coverage
        COMMAND gcovr -r ${CMAKE_CURRENT_LIST_DIR} --filter "${CMAKE_CURRENT_LIST_DIR}/include" -x coverage.info
    )
    add_custom_target(coverage-html
        COMMAND gcovr -r ${CMAKE_CURRENT_LIST_DIR} --filter "${CMAKE_CURRENT_LIST_DIR}/include" --html-details ${CMAKE_BINARY_DIR}/report/report.html
    )
endif ()

if (CACHES_INSTALL_LIBRARY)
    set(INSTALL_GTEST OFF CACHE STRING "")
endif ()

if (CACHES_BUILD_TEST)
    add_subdirectory(deps/)
    add_subdirectory(test)
endif ()

if (CACHES_INSTALL_LIBRARY) 
    # Install headers
    install(DIRECTORY include/ DESTINATION include)

    # Install target
    install(TARGETS ${PROJECT_NAME} EXPORT cachesTargets)
       
    # Export targets
    install(EXPORT ${PROJECT_NAME}Targets
        FILE ${PROJECT_NAME}Targets.cmake
        NAMESPACE ${PROJECT_NAME}::
        DESTINATION lib/cmake/${PROJECT_NAME}
    )

    # Package config
    include(CMakePackageConfigHelpers)
    write_basic_package_version_file(
        "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
        VERSION ${PROJECT_VERSION}
        COMPATIBILITY SameMajorVersion
    )
	
    configure_package_config_file(
        "${CMAKE_CURRENT_SOURCE_DIR}/cmake/${PROJECT_NAME}Config.cmake.in"
        "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
        INSTALL_DESTINATION lib/cmake/${PROJECT_NAME}
    )

    install(FILES
        "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
        "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
        DESTINATION lib/cmake/${PROJECT_NAME}
    )
endif ()
