#
# Copyright (c) 2026 Nordic Semiconductor
#
# SPDX-License-Identifier: LicenseRef-Nordic-5-Clause
#

set(valid_socs
  nrf9160
  nrf9120
  nrf9230
)

set(valid_float_types
  hard-float
  soft-float
)

set(valid_nrf_modem_variants
  cellular
  dect_phy
  dect
)

if(NOT DEFINED CONFIG_NRF_MODEM_SOC)
  set(CONFIG_NRF_MODEM_SOC "nrf54l" CACHE STRING "SoftDevice Controller library SoC variant")
  set_property(CACHE CONFIG_NRF_MODEM_SOC PROPERTY STRINGS ${valid_socs})
endif()

if(NOT DEFINED CONFIG_NRF_MODEM_FLOAT_TYPE)
  set(CONFIG_NRF_MODEM_FLOAT_TYPE "hard-float" CACHE STRING "SoftDevice Controller library float type")
  set_property(CACHE CONFIG_NRF_MODEM_FLOAT_TYPE PROPERTY STRINGS ${valid_float_types})
endif()

if(NOT DEFINED CONFIG_NRF_MODEM_VARIANT)
  set(CONFIG_NRF_MODEM_VARIANT "multirole" CACHE STRING "SoftDevice Controller library role")
  set_property(CACHE CONFIG_NRF_MODEM_VARIANT PROPERTY STRINGS ${valid_CONFIG_NRF_MODEM_VARIANTs})
endif()

if(NOT DEFINED CONFIG_NRF_MODEM_LOG)
  option(CONFIG_NRF_MODEM_LOG "Enable modem library logging" OFF)
endif()

if(NOT DEFINED CONFIG_NRF_MODEM_LINK_BINARY)
  option(CONFIG_NRF_MODEM_LINK_BINARY "Link modem library binaries" OFF)
endif()

if(NOT CONFIG_NRF_MODEM_SOC IN_LIST valid_socs)
  message(FATAL_ERROR "CONFIG_NRF_MODEM_SOC='${CONFIG_NRF_MODEM_SOC}' is not valid. Must be one of: ${valid_socs}")
endif()

if(NOT CONFIG_NRF_MODEM_FLOAT_TYPE IN_LIST valid_float_types)
  message(FATAL_ERROR "CONFIG_NRF_MODEM_FLOAT_TYPE='${CONFIG_NRF_MODEM_FLOAT_TYPE}' is not valid. "
                      "Must be one of: ${valid_float_types}")
endif()

if(NOT CONFIG_NRF_MODEM_VARIANT IN_LIST valid_nrf_modem_variants)
  message(FATAL_ERROR "CONFIG_NRF_MODEM_VARIANT='${CONFIG_NRF_MODEM_VARIANT}' is not valid. "
                      "Must be one of: ${valid_nrf_modem_variants}")
endif()


if(CONFIG_NRF_MODEM_LINK_BINARY)

  cmake_path(SET lib_path ${CMAKE_CURRENT_SOURCE_DIR})
  cmake_path(APPEND lib_path lib)

  cmake_path(APPEND lib_path ${CONFIG_NRF_MODEM_VARIANT})
  cmake_path(APPEND lib_path ${CONFIG_NRF_MODEM_SOC})
  cmake_path(APPEND lib_path ${CONFIG_NRF_MODEM_FLOAT_TYPE})

  # Logging
  if(CONFIG_NRF_MODEM_LOG)
    cmake_path(APPEND lib_path libmodem_log.a)
  else()
    cmake_path(APPEND lib_path libmodem.a)
  endif()

  # Check if library file exists
  if(NOT EXISTS ${lib_path})
    assert(0 "Binary library not found: '${lib_path}'")
  endif()

  add_library(nrf_modem INTERFACE)

  add_library(nrf_modem_imported STATIC IMPORTED GLOBAL)
  set_target_properties(nrf_modem_imported PROPERTIES IMPORTED_LOCATION lib_path)

    target_include_directories(nrf_modem INTERFACE
      ${CMAKE_CURRENT_SOURCE_DIR}/include
    )

  target_link_libraries(nrf_modem INTERFACE nrf_modem_imported)

  if(${CONFIG_NRF_MODEM_SOC} STREQUAL "nrf9160" OR ${CONFIG_NRF_MODEM_SOC} STREQUAL "nrf9120")
    target_sources(nrf_modem PRIVATE shmem.c)
  endif()

endif()
