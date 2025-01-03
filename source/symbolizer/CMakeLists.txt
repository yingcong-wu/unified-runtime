# Copyright (C) 2024 Intel Corporation
# Part of the Unified-Runtime Project, under the Apache License v2.0 with LLVM Exceptions.
# See LICENSE.TXT
# SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception

# An optional symbolizer based on LLVM-symbolizer

set(symbolizer_sources
    ${CMAKE_CURRENT_SOURCE_DIR}/ur_symbolizer.cpp
)

add_library (ur_symbolizer SHARED ${symbolizer_sources})

add_library(${PROJECT_NAME}::symbolizer ALIAS ur_symbolizer)

target_include_directories(ur_symbolizer PRIVATE ${LLVM_INCLUDE_DIRS})
target_link_libraries(ur_symbolizer PRIVATE LLVMSupport LLVMSymbolize)
# In in-tree build, if LLVM is built with libc++, we also need to build
# symbolizer.cpp with libc++ abi and link libc++ in.
if(NOT UR_STANDALONE_BUILD AND LLVM_LIBCXX_USED)
    execute_process(
        COMMAND ${CMAKE_CXX_COMPILER} --print-file-name=libc++.a
        OUTPUT_VARIABLE LIBCXX_PATH
        OUTPUT_STRIP_TRAILING_WHITESPACE)
    execute_process(
        COMMAND ${CMAKE_CXX_COMPILER} --print-file-name=libc++abi.a
        OUTPUT_VARIABLE LIBCXX_ABI_PATH
        OUTPUT_STRIP_TRAILING_WHITESPACE)
    set_property(SOURCE
        ${symbolizer_sources}
        APPEND_STRING PROPERTY COMPILE_FLAGS
        " -stdlib=libc++ ")
    if(NOT EXISTS ${LIBCXX_PATH} OR NOT EXISTS ${LIBCXX_ABI_PATH})
        message(FATAL_ERROR "libc++ is required but can't find the libraries")
    endif()
    target_link_libraries(ur_symbolizer PRIVATE ${LIBCXX_PATH} ${LIBCXX_ABI_PATH})
endif()
endif()
