# Copyright (c) 2023 Intel Corporation.
# SPDX-License-Identifier: Apache-2.0

cmake_minimum_required(VERSION 3.20.0)
find_package(Zephyr REQUIRED HINTS $ENV{ZEPHYR_BASE})
project(llext_simple_test)

target_sources(app PRIVATE
  src/test_llext_simple.c
)

target_include_directories(app PRIVATE
  ${ZEPHYR_BASE}/include
  ${ZEPHYR_BASE}/kernel/include
  ${ZEPHYR_BASE}/arch/${ARCH}/include
)

if(NOT LOADER_BUILD_ONLY)
# generate extension targets foreach extension given by name
foreach(ext_name hello_world logging relative_jump object syscalls threads_kernel_objects)
  set(ext_src ${PROJECT_SOURCE_DIR}/src/${ext_name}_ext.c)
  set(ext_bin ${ZEPHYR_BINARY_DIR}/${ext_name}.llext)
  set(ext_inc ${ZEPHYR_BINARY_DIR}/include/generated/${ext_name}.inc)
  add_llext_target(${ext_name}_ext
    OUTPUT  ${ext_bin}
    SOURCES ${ext_src}
  )
  generate_inc_file_for_target(app ${ext_bin} ${ext_inc})
endforeach()

# Add a dummy custom processing command to test add_llext_command
get_target_property(proc_in_file hello_world_ext lib_output)
get_target_property(proc_out_file hello_world_ext pkg_input)
add_llext_command(
  TARGET hello_world_ext
  POST_BUILD
  COMMAND echo "dummy patching ${proc_in_file} to create ${proc_out_file}"
  COMMAND ${CMAKE_COMMAND} -E copy ${proc_in_file} ${proc_out_file}
)
endif()
