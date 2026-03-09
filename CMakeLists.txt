cmake_minimum_required(VERSION 3.28)
project(hogl VERSION 3.8.2 DESCRIPTION "HOly Grail of Logging" LANGUAGES C CXX)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Options
option(HOGL_DEBUG "Enable internal HOGL debug" OFF)
option(HOGL_ENABLE_OPENMP "Enable OpenMP unit-test" OFF)
option(HOGL_BUILD_TOOLS "Build tools" ON)
option(HOGL_BUILD_TESTS "Build tests" ON)

if(HOGL_DEBUG)
    add_compile_definitions(HOGL_DEBUG)
endif()

# Threading
set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads REQUIRED)

# Need rt for clock_gettime, dl for dlopen
# Note: clock_gettime might be in glibc now, so rt might not be required, but we'll try to find it
find_library(RT_LIBRARY rt)
if(NOT RT_LIBRARY)
    set(RT_LIBRARY "")
endif()

set(HOGL_VERSION ${PROJECT_VERSION})
set(HOGL_LIBS Threads::Threads ${RT_LIBRARY})

set(HOGL_CXXFLAGS "-pthread")
if(RT_LIBRARY)
    set(HOGL_PC_LIBS "-pthread -lrt")
else()
    set(HOGL_PC_LIBS "-pthread")
endif()

# Installation paths
include(GNUInstallDirs)

add_subdirectory(src)

if(HOGL_BUILD_TOOLS)
    add_subdirectory(tools)
endif()

if(HOGL_BUILD_TESTS)
    enable_testing()
    add_subdirectory(tests)
endif()

# Install headers
install(DIRECTORY include/hogl
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
        COMPONENT libhogl-dev
        FILES_MATCHING PATTERN "*.h" PATTERN "*.hpp")

# pkg-config
# we need to set variables for hogl.pc.in
set(prefix ${CMAKE_INSTALL_PREFIX})
set(exec_prefix "\${prefix}")
set(libdir "\${exec_prefix}/${CMAKE_INSTALL_LIBDIR}")
set(includedir "\${prefix}/${CMAKE_INSTALL_INCLUDEDIR}")
set(PACKAGE_VERSION ${PROJECT_VERSION})
set(VERSION ${PROJECT_VERSION})
set(HOGL_LIBS ${HOGL_PC_LIBS})

configure_file(hogl.pc.in hogl.pc @ONLY)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/hogl.pc
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig
        COMPONENT libhogl-dev)

# CMake config files
include(CMakePackageConfigHelpers)
configure_package_config_file(hogl-config.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/hogl-config.cmake
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/hogl
)
write_basic_package_version_file(
    ${CMAKE_CURRENT_BINARY_DIR}/hogl-config-version.cmake
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY AnyNewerVersion
)

install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/hogl-config.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/hogl-config-version.cmake
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/hogl
    COMPONENT libhogl-dev
)

# Packaging
set(CPACK_PACKAGE_NAME "hogl")
set(CPACK_PACKAGE_VENDOR "Max Krasnyansky")
set(CPACK_PACKAGE_CONTACT "Max Krasnyansky <max.krasnyansky@gmail.com>")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "High-performance, low-overhead logging framework")
set(CPACK_PACKAGE_VERSION ${PROJECT_VERSION})
set(CPACK_PACKAGE_VERSION_MAJOR ${PROJECT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${PROJECT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${PROJECT_VERSION_PATCH})

# We want DEB and RPM
set(CPACK_GENERATOR "DEB;RPM")

# Enable component-based packaging
set(CPACK_DEB_COMPONENT_INSTALL ON)
set(CPACK_RPM_COMPONENT_INSTALL ON)

# Component definitions
set(CPACK_COMPONENTS_ALL libhogl libhogl-dev libhogl-tools)

# libhogl configuration
set(CPACK_DEBIAN_LIBHOGL_PACKAGE_NAME "libhogl")
set(CPACK_RPM_LIBHOGL_PACKAGE_NAME "libhogl")
set(CPACK_COMPONENT_LIBHOGL_DESCRIPTION "High performance logging library shared libraries")

# libhogl-dev configuration
set(CPACK_DEBIAN_LIBHOGL-DEV_PACKAGE_NAME "libhogl-dev")
set(CPACK_RPM_LIBHOGL-DEV_PACKAGE_NAME "libhogl-devel")
set(CPACK_DEBIAN_LIBHOGL-DEV_PACKAGE_DEPENDS "libhogl (= ${PROJECT_VERSION})")
set(CPACK_RPM_LIBHOGL-DEV_PACKAGE_REQUIRES "libhogl = ${PROJECT_VERSION}")
set(CPACK_COMPONENT_LIBHOGL-DEV_DESCRIPTION "HOGL header files and development libraries")

# libhogl-tools configuration
set(CPACK_DEBIAN_LIBHOGL-TOOLS_PACKAGE_NAME "libhogl-tools")
set(CPACK_RPM_LIBHOGL-TOOLS_PACKAGE_NAME "libhogl-tools")
set(CPACK_DEBIAN_LIBHOGL-TOOLS_PACKAGE_DEPENDS "libhogl (= ${PROJECT_VERSION})")
set(CPACK_RPM_LIBHOGL-TOOLS_PACKAGE_REQUIRES "libhogl = ${PROJECT_VERSION}")
set(CPACK_COMPONENT_LIBHOGL-TOOLS_DESCRIPTION "HOGL tools like hogl-recover")

include(CPack)
