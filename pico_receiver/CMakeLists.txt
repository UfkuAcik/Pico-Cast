cmake_minimum_required(VERSION 3.13)
include($ENV{PICO_SDK_PATH}/external/pico_sdk_import.cmake)
project(pico_receiver C CXX ASM)
set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 17)

add_compile_options(-Ofast -ffast-math)

pico_sdk_init()

add_executable(pico_receiver
    main.cpp
    JPEGDEC/src/JPEGDEC.cpp
)

target_include_directories(pico_receiver PRIVATE
    ${CMAKE_CURRENT_LIST_DIR}
    ${CMAKE_CURRENT_LIST_DIR}/JPEGDEC/src
    ${CMAKE_CURRENT_LIST_DIR}/PicoDVI/software/include
)

add_subdirectory(PicoDVI/software/libdvi)

# PicoDVI pin configuration is defined in main.cpp
# Olimex RP2040-PICO-PC uses specific pins for TMDS and CLK.

target_link_libraries(pico_receiver
    pico_stdlib
    pico_multicore
    hardware_pio
    hardware_dma
    hardware_irq
    hardware_clocks
    hardware_vreg
    hardware_pwm
    pico_cyw43_arch_lwip_threadsafe_background
    libdvi
)

pico_enable_stdio_usb(pico_receiver 1)
pico_enable_stdio_uart(pico_receiver 0)
pico_add_extra_outputs(pico_receiver)
