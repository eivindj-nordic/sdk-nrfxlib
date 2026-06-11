#
# Copyright (c) 2018 Nordic Semiconductor
#
# SPDX-License-Identifier: LicenseRef-Nordic-5-Clause
#

# SoftDevice library options (defaults and valid values)

set(valid_socs
  nrf52
  nrf53
  nrf54h
  nrf54ls
  nrf54lm_ns
  nrf54lm
  nrf54lv_ns
  nrf54lv
  nrf54l_ns
  nrf54l
  nrf71
  nrf52_bsim
  nrf53_bsim
  nrf54l_bsim
)

set(valid_float_types
  hard-float
  softfp-float
  soft-float
)

set(valid_sdc_variants
  peripheral
  central
  multirole
)

if(NOT DEFINED SDC_SOC)
  set(SDC_SOC "nrf54l" CACHE STRING "SoftDevice Controller library SoC variant")
  set_property(CACHE SDC_SOC PROPERTY STRINGS ${valid_socs})
endif()

if(NOT DEFINED SDC_FLOAT_TYPE)
  set(SDC_FLOAT_TYPE "hard-float" CACHE STRING "SoftDevice Controller library float type")
  set_property(CACHE SDC_FLOAT_TYPE PROPERTY STRINGS ${valid_float_types})
endif()

if(NOT DEFINED SDC_VARIANT)
  set(SDC_VARIANT "multirole" CACHE STRING "SoftDevice Controller library role")
  set_property(CACHE SDC_VARIANT PROPERTY STRINGS ${valid_sdc_variants})
endif()

if(NOT DEFINED SDC_HEADERS_INCLUDE)
  option(SDC_HEADERS_INCLUDE "Expose softdevice_controller/include to dependents" ON)
endif()

if(NOT DEFINED SDC_HEADERS_INCLUDE_PATH_OVERRIDE)
  option(SDC_HEADERS_INCLUDE_PATH_OVERRIDE "Custom include path (disable auto-include)" OFF)
endif()

# Validation

if(NOT SDC_SOC IN_LIST valid_socs)
  message(FATAL_ERROR "SDC_SOC='${SDC_SOC}' is not valid. Must be one of: ${valid_socs}")
endif()

if(NOT SDC_FLOAT_TYPE IN_LIST valid_float_types)
  message(FATAL_ERROR "SDC_FLOAT_TYPE='${SDC_FLOAT_TYPE}' is not valid. "
                      "Must be one of: ${valid_float_types}")
endif()

if(NOT SDC_VARIANT IN_LIST valid_sdc_variants)
  message(FATAL_ERROR "SDC_VARIANT='${SDC_VARIANT}' is not valid. "
                      "Must be one of: ${valid_sdc_variants}")
endif()

# Resolve library path

set(SDC_LIB_DIR "${CMAKE_CURRENT_SOURCE_DIR}/lib/${SDC_SOC}/${SDC_FLOAT_TYPE}")

if(NOT EXISTS "${SDC_LIB_DIR}/libsoftdevice_controller_${SDC_VARIANT}.a")
  message(FATAL_ERROR "This combination of SoC (${SDC_SOC}), floating point ABI (${SDC_FLOAT_TYPE})"
                      " and variant (${SDC_VARIANT}) is not supported by the SoftDevice Controller "
                      " library.")
endif()

add_library(softdevice_controller INTERFACE)

add_library(softdevice_controller_imported STATIC IMPORTED GLOBAL)
set_target_properties(softdevice_controller_imported PROPERTIES
  IMPORTED_LOCATION ${SDC_LIB_DIR}/libsoftdevice_controller_${SDC_VARIANT}.a
)
target_link_libraries(softdevice_controller_imported INTERFACE mpsl)

if(SDC_HEADERS_INCLUDE AND NOT SDC_HEADERS_INCLUDE_PATH_OVERRIDE)
  target_include_directories(softdevice_controller INTERFACE
    ${CMAKE_CURRENT_SOURCE_DIR}/include
  )
endif()

target_link_libraries(softdevice_controller INTERFACE softdevice_controller_imported)
