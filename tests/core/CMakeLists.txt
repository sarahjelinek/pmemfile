#
# Copyright 2017, Intel Corporation
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions
# are met:
#
#     * Redistributions of source code must retain the above copyright
#       notice, this list of conditions and the following disclaimer.
#
#     * Redistributions in binary form must reproduce the above copyright
#       notice, this list of conditions and the following disclaimer in
#       the documentation and/or other materials provided with the
#       distribution.
#
#     * Neither the name of the copyright holder nor the names of its
#       contributors may be used to endorse or promote products derived
#       from this software without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
# "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
# LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
# A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT
# OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
# SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT
# LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
# DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
# THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
# (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
# OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fno-strict-aliasing")

include_directories(.)
add_library(pmemfile_test STATIC pmemfile_test.c)

function(build_test name srcs)
        add_executable(${name} ${srcs})
        target_link_libraries(${name} -L${CMAKE_BINARY_DIR}/src/libpmemfile-core pmemfile-core pmemfile_test)
        add_dependencies(${name} pmemfile-core_shared)
endfunction()

build_test(file_basic basic/basic.c)
build_test(file_crash crash/crash.c)
build_test(file_dirs dirs/dirs.c)
build_test(file_getdents getdents/getdents.c)
build_test(file_mt mt/mt.c)
build_test(file_openp openp/openp.c)
build_test(file_rw rw/rw.c)
build_test(file_stat stat/stat.c)
build_test(file_symlinks symlinks/symlinks.c)

add_checker_targets("tests-core")

function(add_test_generic name tracer)
        if(${tracer} STREQUAL pmemcheck)
                set(ARGS -DTRACE0=valgrind -DTRACE1=--tool=pmemcheck)
        elseif(${tracer} STREQUAL memcheck)
                set(ARGS -DTRACE0=valgrind -DTRACE1=--tool=memcheck)
        elseif(${tracer} STREQUAL helgrind)
                set(ARGS -DTRACE0=valgrind -DTRACE1=--tool=helgrind)
        elseif(${tracer} STREQUAL drd)
                set(ARGS -DTRACE0=valgrind -DTRACE1=--tool=drd)
        elseif(${tracer} STREQUAL kgdb)
                set(ARGS -DTRACE0=konsole -DTRACE1=-e -DTRACE2=cgdb -DTRACE3=--args)
        elseif(${tracer} STREQUAL none)
        elseif(${tracer} STREQUAL none2)
        elseif(${tracer} STREQUAL none3)
        else()
                message(FATAL_ERROR "Unknown tracer '${tracer}'")
        endif()

        add_test(NAME ${name}_${tracer}
                 COMMAND ${CMAKE_COMMAND} -DTEST_NAME=${name}_${tracer} -DPARENT_DIR=${TEST_DIR}/ -DSRC_DIR=${CMAKE_CURRENT_SOURCE_DIR} ${ARGS} ${ARGV2} -P ${CMAKE_CURRENT_SOURCE_DIR}/${name}/test.cmake)

        set_tests_properties(${name}_${tracer} PROPERTIES
                ENVIRONMENT "LD_LIBRARY_PATH=${CMAKE_BINARY_DIR}/src/libpmemfile-core;${ARGV3}")
endfunction()

add_test_generic(basic none)
add_test_generic(basic pmemcheck)
add_test_generic(basic memcheck)
add_test_generic(basic helgrind)

add_test_generic(crash none)

add_test_generic(dirs none)
add_test_generic(dirs pmemcheck)
add_test_generic(dirs memcheck)

add_test_generic(getdents none)
add_test_generic(getdents pmemcheck)
add_test_generic(getdents memcheck)

add_test_generic(mt none      -Dops=10000)
add_test_generic(mt pmemcheck -Dops=50)
add_test_generic(mt memcheck  -Dops=10)
add_test_generic(mt helgrind  -Dops=20)
add_test_generic(mt drd       -Dops=20)

add_test_generic(openp none)
add_test_generic(openp memcheck)

add_test_generic(rw none)
add_test_generic(rw none2 '' PMEMFILECORE_BLOCK_SIZE=4096)
add_test_generic(rw memcheck)
add_test_generic(rw pmemcheck)

add_test_generic(stat none)
add_test_generic(stat memcheck)
add_test_generic(stat pmemcheck)

add_test_generic(symlinks none)
add_test_generic(symlinks memcheck)