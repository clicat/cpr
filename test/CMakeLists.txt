cmake_minimum_required(VERSION 3.15)

find_package(Threads REQUIRED)

if (ENABLE_SSL_TESTS)
    add_library(test_server STATIC
                abstractServer.cpp
                httpServer.cpp
                httpsServer.cpp
                testUtils.cpp)
else ()
    add_library(test_server STATIC
                abstractServer.cpp
                httpServer.cpp
                testUtils.cpp)
endif()
if(WIN32)
    target_link_libraries(test_server PRIVATE Threads::Threads cpr::cpr GTest::GTest
                                      PUBLIC mongoose ws2_32 wsock32)
else()
    target_link_libraries(test_server PRIVATE Threads::Threads cpr::cpr GTest::GTest
                                      PUBLIC mongoose)
endif()

macro(add_cpr_test _TEST_NAME)
    add_executable(${_TEST_NAME}_tests ${_TEST_NAME}_tests.cpp)
    target_link_libraries(${_TEST_NAME}_tests PRIVATE
        test_server
        GTest::GTest
        cpr::cpr
        ${CURL_LIB})
    add_test(NAME cpr_${_TEST_NAME}_tests COMMAND ${_TEST_NAME}_tests)
    # Group under the "tests" project folder in IDEs such as Visual Studio.
    set_property(TARGET ${_TEST_NAME}_tests PROPERTY FOLDER "tests")
    if(WIN32 AND BUILD_SHARED_LIBS)
        add_custom_command(TARGET ${_TEST_NAME}_tests POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:libcurl> $<TARGET_FILE_DIR:${_TEST_NAME}_tests>)
        add_custom_command(TARGET ${_TEST_NAME}_tests POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:cpr> $<TARGET_FILE_DIR:${_TEST_NAME}_tests>)
    endif()
endmacro()

add_cpr_test(get)
add_cpr_test(post)
add_cpr_test(session)
add_cpr_test(prepare)
add_cpr_test(async)
if(CPR_BUILD_TESTS_PROXY)
    add_cpr_test(proxy)
    add_cpr_test(proxy_auth)
endif()
add_cpr_test(head)
add_cpr_test(delete)
add_cpr_test(put)
add_cpr_test(callback)
add_cpr_test(raw_body)
add_cpr_test(options)
add_cpr_test(patch)
add_cpr_test(error)
add_cpr_test(alternating)
add_cpr_test(util)
add_cpr_test(structures)
add_cpr_test(encoded_auth)
add_cpr_test(version)
add_cpr_test(download)
add_cpr_test(interceptor)
add_cpr_test(interceptor_multi)
add_cpr_test(multiperform)
add_cpr_test(resolve)
add_cpr_test(multiasync)
add_cpr_test(file_upload)
add_cpr_test(singleton)
add_cpr_test(threadpool)
add_cpr_test(testUtils)
add_cpr_test(connection_pool)

if (ENABLE_SSL_TESTS)
    add_cpr_test(ssl)

    # Install all ssl keys and certs. Explicit copy for each file to prevent issues when copying on Windows.
    add_custom_command(TARGET ssl_tests POST_BUILD COMMAND ${CMAKE_COMMAND} -E make_directory $<TARGET_FILE_DIR:ssl_tests>/data/certificates $<TARGET_FILE_DIR:ssl_tests>/data/keys)
    add_custom_command(TARGET ssl_tests POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_if_different ${CMAKE_CURRENT_SOURCE_DIR}/data/certificates/client.crt $<TARGET_FILE_DIR:ssl_tests>/data/certificates/client.crt)
    add_custom_command(TARGET ssl_tests POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_if_different ${CMAKE_CURRENT_SOURCE_DIR}/data/certificates/root-ca.crt $<TARGET_FILE_DIR:ssl_tests>/data/certificates/root-ca.crt)
    add_custom_command(TARGET ssl_tests POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_if_different ${CMAKE_CURRENT_SOURCE_DIR}/data/certificates/sub-ca.crt $<TARGET_FILE_DIR:ssl_tests>/data/certificates/sub-ca.crt)
    add_custom_command(TARGET ssl_tests POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_if_different ${CMAKE_CURRENT_SOURCE_DIR}/data/certificates/ca-bundle.crt $<TARGET_FILE_DIR:ssl_tests>/data/certificates/ca-bundle.crt)
    add_custom_command(TARGET ssl_tests POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_if_different ${CMAKE_CURRENT_SOURCE_DIR}/data/certificates/server.crt $<TARGET_FILE_DIR:ssl_tests>/data/certificates/server.crt)
    add_custom_command(TARGET ssl_tests POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_if_different ${CMAKE_CURRENT_SOURCE_DIR}/data/keys/client.key $<TARGET_FILE_DIR:ssl_tests>/data/keys/client.key)
    add_custom_command(TARGET ssl_tests POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_if_different ${CMAKE_CURRENT_SOURCE_DIR}/data/keys/root-ca.key $<TARGET_FILE_DIR:ssl_tests>/data/keys/root-ca.key)
    add_custom_command(TARGET ssl_tests POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_if_different ${CMAKE_CURRENT_SOURCE_DIR}/data/keys/server.key $<TARGET_FILE_DIR:ssl_tests>/data/keys/server.key)
    add_custom_command(TARGET ssl_tests POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_if_different ${CMAKE_CURRENT_SOURCE_DIR}/data/keys/server.pub $<TARGET_FILE_DIR:ssl_tests>/data/keys/server.pub)
endif()

add_custom_command(TARGET file_upload_tests POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_if_different ${CMAKE_CURRENT_SOURCE_DIR}/data/test_file_hello_äüöp_2585_你好.txt $<TARGET_FILE_DIR:file_upload_tests>/data/test_file_hello_äüöp_2585_你好.txt)
add_custom_command(TARGET file_upload_tests POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_if_different ${CMAKE_CURRENT_SOURCE_DIR}/data/test_file_hello_äüöp_2585.txt $<TARGET_FILE_DIR:file_upload_tests>/data/test_file_hello_äüöp_2585.txt)
add_custom_command(TARGET file_upload_tests POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_if_different ${CMAKE_CURRENT_SOURCE_DIR}/data/test_file.txt $<TARGET_FILE_DIR:file_upload_tests>/data/test_file.txt)

file(INSTALL data DESTINATION data)
