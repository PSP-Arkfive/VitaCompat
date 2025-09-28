cmake_minimum_required(VERSION 3.28)

project(vitacompat VERSION 1.0 LANGUAGES C ASM)

set(DEBUG "" CACHE STRING "Debug level (1,2,3)")
message(STATUS "DEBUG LEVEL = ${DEBUG}")

include(FetchContent)
FetchContent_Declare(
    arkdevsdk
    GIT_REPOSITORY https://github.com/PSP-Arkfive/ark-dev-sdk.git
    GIT_TAG main
    EXCLUDE_FROM_ALL
)
FetchContent_Declare(
    bootloadex
    GIT_REPOSITORY https://github.com/PSP-Arkfive/BootLoadEx.git
    GIT_TAG main
    EXCLUDE_FROM_ALL
)
FetchContent_MakeAvailable(arkdevsdk bootloadex)

set(ARKSDK ${arkdevsdk_SOURCE_DIR})

add_prx_module(vitacompat exports.exp)

target_sources(vitacompat PRIVATE 
   main.c
   src/vlf.c
   src/syspatch.c
   src/vitamem.c
   src/vitaflash.c 
   src/popspatch.c
   src/filesystem.c
   src/fatef.c
   src/flashfs.c
   imports.S
   payload.h
)

remove_definitions("-D_PSP_FW_VERSION=600")
add_definitions("-D_PSP_FW_VERSION=660")

target_compile_options(vitacompat PRIVATE -std=c99 -Os -G0 -Wall -fno-pic)
target_include_directories(vitacompat PRIVATE include ${ARKSDK}/include ${CMAKE_CURRENT_BINARY_DIR})
target_link_directories(vitacompat PRIVATE ${ARKSDK}/libs)
target_link_libraries(vitacompat PRIVATE
   -nostartfiles
   -nostdlib
   pspsystemctrl_kernel
   pspkermit_driver
   pspkermitperipheral_driver
   pspsdk
   pspmodinfo
   pspkernel
)

add_custom_command(OUTPUT payload.h
  DEPENDS vitacompat_reboot
  COMMAND bin2c "$<TARGET_FILE:vitacompat_reboot>.bin.gz" payload.h rebootbuffer_vita
)

add_subdirectory(rebootex)