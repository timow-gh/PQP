cmake_minimum_required(VERSION 3.21)

project(pqp)

set(VERSION_MAJOR 1)
set(VERSION_MINOR 3)
set(VERSION ${VERSION_MAJOR}.${VERSION_MINOR})

if ((CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR))
    set(IS_TOP_LEVEL_PROJECT TRUE)
else ()
    set(IS_TOP_LEVEL_PROJECT FALSE)
endif ()

if (IS_TOP_LEVEL_PROJECT)
    option(${PROJECT_NAME}_INSTALL "Install project targets" ON)
    option(${PROJECT_NAME}_BUILD_SAMPLES "Build samples" ON)
    option(${PROJECT_NAME}_PACKAGE "Use CPack to create an installer" ON)
else ()
    option(${PROJECT_NAME}_INSTALL "Install project targets" OFF)
    option(${PROJECT_NAME}_BUILD_SAMPLES "Build samples" OFF)
    option(${PROJECT_NAME}_PACKAGE "Use CPack to create an installer" OFF)
endif ()

set(
        HDRS
        src/BV.h
        src/PQP.h
        src/PQP_Compile.h
        src/PQP_Internal.h
        src/Tri.h
)

set(
        SRCS
        src/Build.cpp
        src/BV.cpp
        src/PQP.cpp
        src/TriDist.cpp
)

add_library(PQP STATIC ${HDRS} ${SRCS})

target_include_directories(
        PQP
        INTERFACE
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
        $<INSTALL_INTERFACE:$<INSTALL_PREFIX>/${CMAKE_INSTALL_INCLUDEDIR}>
)

set_target_properties(
        PQP
        PROPERTIES
        DEBUG_POSTFIX d
        POSITION_INDEPENDENT_CODE ON
        VERSION ${VERSION}
)

if (${PROJECT_NAME}_BUILD_SAMPLES)
    add_subdirectory(demo_dependencies)

    add_executable(sample demos/sample/main.cpp)
    target_link_libraries(sample PQP)

    configure_file(demos/spinning/bunny.tris bunny.tris COPYONLY)
    configure_file(demos/spinning/torus.tris torus.tris COPYONLY)
    add_executable(spinning demos/spinning/main.cpp demos/spinning/model.cpp)
    target_link_libraries(spinning PRIVATE PQP freeglut_static)

    configure_file(demos/falling/torus1.path torus1.path COPYONLY)
    configure_file(demos/falling/torus1.tris torus1.tris COPYONLY)
    configure_file(demos/falling/torus2.path torus2.path COPYONLY)
    configure_file(demos/falling/torus2.tris torus2.tris COPYONLY)
    add_executable(falling demos/falling/main.cpp demos/falling/model.cpp)
    target_link_libraries(falling PRIVATE PQP freeglut_static)
endif ()

if (${PROJECT_NAME}_INSTALL)
    include(GNUInstallDirs)
    install(FILES ${HDRS} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR} COMPONENT development)

    install(
            TARGETS PQP
            EXPORT PQP
            ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT development
            LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT runtime NAMELINK_SKIP
            RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT runtime
    )

    configure_package_config_file(
            pqp-config.cmake.in pqp-config.cmake
            INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/PQP-${VERSION}
    )

    install(
            FILES ${CMAKE_CURRENT_BINARY_DIR}/pqp-config.cmake
            DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/PQP-${VERSION}
            COMPONENT development
    )

    write_basic_package_version_file(
            ${CMAKE_CURRENT_BINARY_DIR}/pqp-config-version.cmake
            VERSION ${VERSION}
            COMPATIBILITY ExactVersion
    )

    install(
            FILES ${CMAKE_CURRENT_BINARY_DIR}/pqp-config-version.cmake
            DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/PQP-${VERSION}
            COMPONENT development
    )

    install(
            EXPORT PQP
            DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/PQP-${VERSION}
            NAMESPACE PQP::
            FILE pqp-export.cmake
            COMPONENT development
    )
endif ()

if (${PROJECT_NAME}_PACKAGE)
    include(CMakePackageConfigHelpers)
    include(CPack)

    set(CPACK_COMPONENT_DEVELOPMENT_DEPENDS runtime)
    set(CPACK_DEB_COMPONENT_INSTALL ON)
    set(CPACK_DEBIAN_DEVELOPMENT_PACKAGE_DEPENDS "libpqp")
    set(CPACK_DEBIAN_DEVELOPMENT_PACKAGE_NAME "libpqp-dev")
    set(CPACK_DEBIAN_DEVELOPMENT_PACKAGE_SECTION "libdevel")
    set(CPACK_DEBIAN_PACKAGE_HOMEPAGE "http://gamma.cs.unc.edu/SSV/")
    set(CPACK_DEBIAN_PACKAGE_NAME "libpqp")
    set(CPACK_DEBIAN_PACKAGE_SECTION "devel")
    set(CPACK_DEBIAN_RUNTIME_PACKAGE_NAME "libpqp")
    set(CPACK_DEBIAN_RUNTIME_PACKAGE_SECTION "libs")
    set(CPACK_NSIS_PACKAGE_NAME "PQP ${VERSION}")
    set(CPACK_NSIS_URL_INFO_ABOUT ${CPACK_DEBIAN_PACKAGE_HOMEPAGE})
    set(CPACK_PACKAGE_CONTACT "geom@cs.unc.edu")
    set(CPACK_PACKAGE_DISPLAY_NAME "PQP ${VERSION}")
    set(CPACK_PACKAGE_INSTALL_DIRECTORY "PQP-${VERSION}")
    set(CPACK_PACKAGE_NAME "pqp")
    set(CPACK_PACKAGE_VERSION ${VERSION})
    set(CPACK_PACKAGE_VERSION_MAJOR ${VERSION_MAJOR})
    set(CPACK_PACKAGE_VERSION_MINOR ${VERSION_MINOR})
    set(CPACK_RESOURCE_FILE_LICENSE ${CMAKE_CURRENT_SOURCE_DIR}/LICENSE)
    set(CPACK_RPM_COMPONENT_INSTALL ON)
    set(CPACK_RPM_development_PACKAGE_NAME "pqp-devel")
    set(CPACK_RPM_development_PACKAGE_REQUIRES "pqp")
    set(CPACK_RPM_PACKAGE_GROUP "Development/Libraries")
    set(CPACK_RPM_PACKAGE_NAME "pqp")
    set(CPACK_RPM_PACKAGE_URL ${CPACK_DEBIAN_PACKAGE_HOMEPAGE})
    set(CPACK_RPM_runtime_PACKAGE_NAME "pqp")

    if (WIN32 AND CMAKE_SIZEOF_VOID_P EQUAL 8)
        set(CPACK_NSIS_INSTALL_ROOT "$PROGRAMFILES64")
    endif ()

endif ()
