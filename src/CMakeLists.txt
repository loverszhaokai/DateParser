cmake_minimum_required(VERSION 3.3)
project(src)

include_directories(./ ${CMAKE_CURRENT_BINARY_DIR} ../third_party/googletest/googletest/include/)

add_definitions(-D__STDC_CONSTANT_MACROS
        -D__STDC_FORMAT_MACROS)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} \
    -fno-threadsafe-statics -std=c++11 -fvisibility-inlines-hidden \
    -std=gnu++11 -m64 -march=x86-64 -fvisibility=hidden -fPIC")

set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -O0 -g -D_DEBUG")

set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -O2 \
    -DDYNAMIC_ANNOTATIONS_ENABLED=0 -DNDEBUG -DNVALGRIND")

set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ")

IF(CMAKE_BUILD_ASAN_TYPE MATCHES 1)
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} \
        -fsanitize=address -fno-omit-frame-pointer")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -lasan")
ENDIF(CMAKE_BUILD_ASAN_TYPE MATCHES 1)

#add_library(date_parser date_parser.cc date_parser.h)
#add_executable(date_parser_unittest date_parser_unittest.cc)
#target_link_libraries(date_parser_unittest gtest_main date_parser)

BISON_TARGET(mc_parser mc_parser.yy ${CMAKE_CURRENT_BINARY_DIR}/mc_parser.cc)
FLEX_TARGET(mc_lexer mc_lexer.l ${CMAKE_CURRENT_BINARY_DIR}/mc_lexer.cc)

ADD_FLEX_BISON_DEPENDENCY(mc_lexer mc_parser)

set(SOURCE_FILES
    mc_driver.cc
    mc_driver.h
    mc_scanner.h)

add_library(date_parser ${SOURCE_FILES}
            ${BISON_mc_parser_OUTPUTS}
            ${FLEX_mc_lexer_OUTPUTS})

add_executable(date_parser_unittest date_parser_test.cc test.cc
               time_util.cc time_util.h test.cc test.h)

target_link_libraries(date_parser_unittest date_parser)
