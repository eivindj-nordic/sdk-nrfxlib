#
# Copyright (c) 2026 Nordic Semiconductor
#
# SPDX-License-Identifier: LicenseRef-Nordic-5-Clause
#

# MPSL library options (defaults and valid values)

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

if(NOT DEFINED MPSL_SOC)
  set(MPSL_SOC "nrf54l" CACHE STRING "MPSL library SoC variant")
  set_property(CACHE MPSL_SOC PROPERTY STRINGS ${valid_socs})
endif()

if(NOT DEFINED MPSL_FLOAT_TYPE)
set(MPSL_FLOAT_TYPE "hard-float" CACHE STRING "MPSL library float type")
set_property(CACHE MPSL_FLOAT_TYPE PROPERTY STRINGS ${valid_float_types})
endif()

# Validation

if(NOT MPSL_SOC IN_LIST valid_socs)
  message(FATAL_ERROR "MPSL_SOC='${MPSL_SOC}' is not valid. Must be one of: ${valid_socs}")
endif()

if(NOT MPSL_FLOAT_TYPE IN_LIST valid_float_types)
  message(FATAL_ERROR "MPSL_FLOAT_TYPE='${MPSL_FLOAT_TYPE}' is not valid. "
                      "Must be one of: ${valid_float_types}")
endif()

# Resolve library path

set(MPSL_LIB_DIR "${CMAKE_CURRENT_SOURCE_DIR}/lib/${MPSL_SOC}/${MPSL_FLOAT_TYPE}")

if(NOT EXISTS "${MPSL_LIB_DIR}/libmpsl.a")
  message(FATAL_ERROR "This combination of SoC (${MPSL_SOC}) and floating point ABI "
                      "(${MPSL_FLOAT_TYPE}) is not supported by the MPSL library.")
endif()

# Define library target

add_library(mpsl INTERFACE)

add_subdirectory(fem)

add_library(mpsl_imported STATIC IMPORTED GLOBAL)
set_target_properties(mpsl_imported PROPERTIES
  IMPORTED_LOCATION ${MPSL_LIB_DIR}/libmpsl.a
)
target_include_directories(mpsl INTERFACE
  ${CMAKE_CURRENT_SOURCE_DIR}/include
  ${CMAKE_CURRENT_SOURCE_DIR}/include/protocol
)

target_link_libraries(mpsl INTERFACE mpsl_imported)
