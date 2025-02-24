cmake_minimum_required(VERSION 3.16)

# Project name
project(blink C ASM)

# ✅ Define CPU, FPU, and Float ABI correctly
set(CPU_FLAGS -mcpu=cortex-m4 -mthumb -mfpu=fpv4-sp-d16 -mfloat-abi=softfp)
set(COMPILE_OPTIONS -Wall -ffunction-sections -fdata-sections -g -O2)
set(LINKER_OPTIONS -Wl,--gc-sections --specs=nosys.specs)

# ✅ Set compiler
set(CMAKE_SYSTEM_NAME Generic)
set(CMAKE_SYSTEM_PROCESSOR arm)

set(CMAKE_C_COMPILER arm-none-eabi-gcc)
set(CMAKE_ASM_COMPILER arm-none-eabi-gcc)

# ✅ Linker script
set(LINKER_SCRIPT ${CMAKE_SOURCE_DIR}/Debug/STM32F411RETX_FLASH.ld)

# ✅ Include directories (Matches CubeIDE)
include_directories(
    Core/Inc
    Drivers/CMSIS/Core/Include
    Drivers/CMSIS/Device/ST/STM32F4xx/Include
    Drivers/STM32F4xx_HAL_Driver/Inc
)

# ✅ Source files (Including LL Drivers)
set(SRCS
    Core/Src/main.c
    Core/Src/stm32f4xx_it.c
    Core/Src/syscalls.c
    Core/Src/sysmem.c
    Core/Src/system_stm32f4xx.c
    Core/Startup/startup_stm32f411retx.s
    # ✅ Add all LL driver source files (Fixes undefined references)
    Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_ll_gpio.c
    Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_ll_exti.c
    Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_ll_usart.c
    Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_ll_rcc.c
    Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_ll_utils.c
    Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_ll_pwr.c
)

# ✅ Define Executable
add_executable(blink.elf ${SRCS})

# ✅ Compiler & Linker flags applied properly
target_compile_options(blink.elf PRIVATE ${CPU_FLAGS} ${COMPILE_OPTIONS})
target_link_options(blink.elf PRIVATE ${CPU_FLAGS} ${LINKER_OPTIONS} -T${LINKER_SCRIPT} -Wl,-Map=blink.map)

# ✅ Define macros
target_compile_definitions(blink.elf PRIVATE USE_FULL_LL_DRIVER)

# ✅ Post-build: Generate HEX & BIN files
add_custom_command(TARGET blink.elf POST_BUILD
    COMMAND arm-none-eabi-size $<TARGET_FILE:blink.elf>
)

add_custom_command(TARGET blink.elf POST_BUILD
    COMMAND arm-none-eabi-objcopy -O ihex $<TARGET_FILE:blink.elf> blink.hex
)

add_custom_command(TARGET blink.elf POST_BUILD
    COMMAND arm-none-eabi-objcopy -O binary $<TARGET_FILE:blink.elf> blink.bin
)
