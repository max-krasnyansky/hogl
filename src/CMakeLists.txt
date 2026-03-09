add_library(hogl SHARED
    fmt-format.cc
    area.cc
    internal.cc
    ringbuf.cc
    tls.cc
    engine.cc
    timesource.cc
    platform.cc
    schedparam.cc
    post.cc
    mask.cc
    flush.cc
    output.cc
    ostrbuf.cc
    ostrbuf-fd.cc
    ostrbuf-stdio.cc
    ostrbuf-tee.cc
    format-basic.cc
    format-raw.cc
    output-stdout.cc
    output-stderr.cc
    output-plainfile.cc
    output-file.cc
    output-pipe.cc
    output-tee.cc
    default-ringopts.cc
    default-timesource.cc
    default-engine.cc
    c-api.cc
)

target_include_directories(hogl PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/../include>
    $<INSTALL_INTERFACE:include>
)

target_link_libraries(hogl PUBLIC Threads::Threads)
if(RT_LIBRARY)
    target_link_libraries(hogl PRIVATE ${RT_LIBRARY})
endif()

set_target_properties(hogl PROPERTIES
    VERSION ${HOGL_VERSION}
    SOVERSION ${PROJECT_VERSION_MAJOR}
)

install(TARGETS hogl
    EXPORT hogl-targets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

install(EXPORT hogl-targets
    FILE hogl-targets.cmake
    NAMESPACE hogl::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/hogl
)
