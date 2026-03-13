find_package(Boost REQUIRED COMPONENTS unit_test_framework)

set(TESTS
    schedparam_test
    args_test
    area_test
    mask_test
    ring_test
    tso_test
    engine_test
    min_test
    basic_test
    aux_engine_test
    rawrec_test
    defring_test
    stress_test
    ooo_test
    untouch_test
    c-api_test
    basic_test_c-api
    basic_test_c-api_cc
    format_basic_test
    ostrbuf_test
)

foreach(test_name IN LISTS TESTS)
    # Check if there is a specific source file, else use ${test_name}.cc
    if(test_name STREQUAL "basic_test_c-api")
        add_executable(${test_name} basic_test_c-api.c dummy.cc) # dummy.cc to force CXX link
    elseif(test_name STREQUAL "basic_test_c-api_cc")
        add_executable(${test_name} basic_test_c-api_cc.cc)
    else()
        add_executable(${test_name} ${test_name}.cc)
    endif()

    target_link_libraries(${test_name} PRIVATE hogl Boost::unit_test_framework)
    add_test(NAME ${test_name} COMMAND ${test_name})
endforeach()

if(HOGL_ENABLE_OPENMP)
    find_package(OpenMP REQUIRED)
    add_executable(openmp_test openmp_test.cc)
    target_link_libraries(openmp_test PRIVATE hogl OpenMP::OpenMP_CXX Boost::unit_test_framework)
    add_test(NAME openmp_test COMMAND openmp_test)
endif()

# Tests disasm
add_executable(disasm_test
    disasm/record.cc
    disasm/post.cc
    disasm/c-api-post.c
    disasm/bitmap.cc
    disasm/ring.cc
    disasm/tls.cc
    disasm/main.cc
)
target_link_libraries(disasm_test PRIVATE hogl Boost::unit_test_framework)
add_test(NAME disasm_test COMMAND disasm_test)

# Plugin test
add_library(fmt-plugin_test MODULE fmt-plugin_test.cc)
target_link_libraries(fmt-plugin_test PRIVATE hogl Boost::unit_test_framework)
