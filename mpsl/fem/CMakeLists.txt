#
# Copyright (c) 2026 Nordic Semiconductor
#
# SPDX-License-Identifier: LicenseRef-Nordic-5-Clause
#

function(mpsl_fem_append_lib fem_lib_name)
  set(LIB_DIR "${CMAKE_CURRENT_SOURCE_DIR}/${fem_lib_name}/lib/${MPSL_SOC}/${MPSL_FLOAT_TYPE}")

  if(IS_DIRECTORY ${LIB_DIR})
    add_library(mpsl_fem_${fem_lib_name}_imported STATIC IMPORTED GLOBAL)
    set_target_properties(mpsl_fem_${fem_lib_name}_imported PROPERTIES
      IMPORTED_LOCATION ${LIB_DIR}/libmpsl_fem_${fem_lib_name}.a
    )
    target_include_directories(mpsl_fem_${fem_lib_name}_imported INTERFACE ${fem_lib_name}/include)

    target_link_libraries(mpsl INTERFACE mpsl_fem_${fem_lib_name}_imported)
  endif()
endfunction()

file(GLOB sub_dir_list RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/*)

foreach(sub_dir ${sub_dir_list})
  if((IS_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/${sub_dir}) AND NOT (${sub_dir} STREQUAL "include"))
    mpsl_fem_append_lib(${sub_dir})
  endif()
endforeach()

target_include_directories(mpsl INTERFACE
  ${CMAKE_CURRENT_SOURCE_DIR}/include
  ${CMAKE_CURRENT_SOURCE_DIR}/include/protocol
)
