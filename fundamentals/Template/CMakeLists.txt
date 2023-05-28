cmake_minimum_required(VERSION 3.22)

#
# Core project settings
#
project(LEDBlinking)          # Modified
enable_language(C CXX ASM)
message("Build type: " ${CMAKE_BUILD_TYPE})

# Setup compiler settings
set(CMAKE_C_STANDARD                11)
set(CMAKE_C_STANDARD_REQUIRED       ON)
set(CMAKE_C_EXTENSIONS              ON)
set(CMAKE_CXX_STANDARD              20)
set(CMAKE_CXX_STANDARD_REQUIRED     ON)
set(CMAKE_CXX_EXTENSIONS            ON)

set(PROJ_PATH                       ${CMAKE_CURRENT_SOURCE_DIR})


#
# Core MCU flags, CPU, instruction set and FPU setup
# Needs to be set properly for your MCU
#
set(CPU_PARAMETERS
    # -mcpu=cortex-m4                 # Modified
    -mfpu=fpv4-sp-d16               # Modified
    -mfloat-abi=hard                # Modified
    -mthumb
)

# Set linker script
set(linker_script_SRC               ${PROJ_PATH}/STM32F469NIHX_FLASH.ld) # Modified
set(EXECUTABLE                      ${CMAKE_PROJECT_NAME})

#
# List of source files to compile
#
set(sources_SRCS                    # Modified
    ${PROJ_PATH}/Core/Src/main.c
    ${PROJ_PATH}/Core/Src/stm32f4xx_hal_msp.c
    ${PROJ_PATH}/Core/Src/stm32f4xx_hal_timebase_tim.c
    ${PROJ_PATH}/Core/Src/stm32f4xx_it.c
    ${PROJ_PATH}/Core/Src/syscalls.c
    ${PROJ_PATH}/Core/Src/sysmem.c
    ${PROJ_PATH}/Core/Src/system_stm32f4xx.c
    ${PROJ_PATH}/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal.c
    ${PROJ_PATH}/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_cortex.c
    ${PROJ_PATH}/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_dma.c
    ${PROJ_PATH}/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_dma_ex.c
    ${PROJ_PATH}/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_exti.c
    ${PROJ_PATH}/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_flash.c
    ${PROJ_PATH}/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_flash_ex.c
    ${PROJ_PATH}/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_flash_ramfunc.c
    ${PROJ_PATH}/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_gpio.c
    ${PROJ_PATH}/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_pwr.c
    ${PROJ_PATH}/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_pwr_ex.c
    ${PROJ_PATH}/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_rcc.c
    ${PROJ_PATH}/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_rcc_ex.c
    ${PROJ_PATH}/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_tim.c
    ${PROJ_PATH}/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_tim_ex.c
    ${PROJ_PATH}/Drivers/STM32F4xx_HAL_Driver/Src/
    ${PROJ_PATH}/Drivers/STM32F4xx_HAL_Driver/Src/
    ${PROJ_PATH}/Drivers/STM32F4xx_HAL_Driver/Src/
)

#
# Include directories
#
set(include_path_DIRS               # Modified
    ${PROJ_PATH}/Core/Inc
    ${PROJ_PATH}/Drivers/STM32F4xx_HAL_Driver/Inc
    ${PROJ_PATH}/Drivers/STM32F4xx_HAL_Driver/Inc/Legacy
    ${PROJ_PATH}/Drivers/CMSIS/Device/ST/STM32F4xx/Include
    ${PROJ_PATH}/Drivers/CMSIS/Include
)

#
# Symbols definition
#
set(symbols_SYMB                    # Modified
    "DEBUG"
    "USE_HAL_DRIVER"
    "STM32F469xx"
)

# Link directories setup
# Must be before executable is added
# link_directories(${EXECUTABLE} ${link_DIRS})

# Executable files
add_executable(${EXECUTABLE} ${sources_SRCS})
# Add sources to executable
target_sources(${EXECUTABLE} PUBLIC ${sources_SRCS})

# Include paths
target_include_directories(${EXECUTABLE} PRIVATE ${include_path_DIRS})

# Project symbols
target_compile_definitions(${EXECUTABLE} PRIVATE ${symbols_SYMB})

# Add linked libraries
# target_link_libraries(${EXECUTABLE} ${link_LIBS})

# Compiler options
target_compile_options(${EXECUTABLE} PRIVATE
    -O0 -ffunction-sections -fdata-sections -Wall -fstack-usage -fcyclomatic-complexity --specs=nano.specs -mfpu=fpv4-sp-d16 -mfloat-abi=hard -mthumb
)

# Linker options
target_link_options(${EXECUTABLE} PRIVATE
    -mcpu=cortex-m4 
    -T${linker_script_SRC}
    --specs=nosys.specs
    -Wl,-Map=${CMAKE_PROJECT_NAME}.map
    -Wl,--gc-sections -static --specs=nano.specs -mfpu=fpv4-sp-d16 -mfloat-abi=hard -mthumb
    -Wl,--start-group -lc -lm -Wl,--end-group
    -Wl,--print-memory-usage
)

# Execute post-build to print size
add_custom_command(TARGET ${EXECUTABLE} POST_BUILD
    COMMAND ${CMAKE_SIZE} $<TARGET_FILE:${EXECUTABLE}>
)

# Convert output to hex and binary
add_custom_command(TARGET ${EXECUTABLE} POST_BUILD
    COMMAND ${CMAKE_OBJCOPY} -O ihex $<TARGET_FILE:${EXECUTABLE}> ${EXECUTABLE}.hex
)

# Convert to bin file -> add conditional check?
add_custom_command(TARGET ${EXECUTABLE} POST_BUILD
    COMMAND ${CMAKE_OBJCOPY} -O binary $<TARGET_FILE:${EXECUTABLE}> ${EXECUTABLE}.bin
)
