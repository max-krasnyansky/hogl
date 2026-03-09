find_package(PkgConfig)
# We need bfd, inotify, dl
find_library(BFD_LIBRARY bfd)
find_library(DL_LIBRARY dl)
find_library(INOTIFY_LIBRARY inotify)

if(NOT BFD_LIBRARY)
    message(FATAL_ERROR "Couldn't find bfd library, please install binutils-dev")
endif()

if(NOT DL_LIBRARY)
    set(DL_LIBRARY "")
endif()

if(NOT INOTIFY_LIBRARY)
    set(INOTIFY_LIBRARY "")
endif()

add_executable(hogl-recover
    coredump.cc
    recovery-engine.cc
    recover.cc
)
target_link_libraries(hogl-recover PRIVATE hogl ${BFD_LIBRARY} ${DL_LIBRARY} ${INOTIFY_LIBRARY})

add_executable(hogl-cook
    rdbuf.cc
    raw-parser.cc
    cook.cc
)
target_link_libraries(hogl-cook PRIVATE hogl)

install(TARGETS hogl-recover hogl-cook
    DESTINATION ${CMAKE_INSTALL_BINDIR}
    COMPONENT libhogl-tools
)

install(PROGRAMS hogl-grep hogl-trace
    DESTINATION ${CMAKE_INSTALL_BINDIR}
    COMPONENT libhogl-tools
)
target_include_directories(hogl-recover PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/..)
target_include_directories(hogl-cook PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/..)
