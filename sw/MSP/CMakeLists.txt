cmake_minimum_required(VERSION 3.10)

# Set the project name
project(enclosure-system)

# Specify the cross-compiler
set(CMAKE_SYSTEM_NAME Generic)
set(CMAKE_SYSTEM_PROCESSOR cortex-m4)
set(CMAKE_C_COMPILER arm-none-eabi-gcc)
set(CMAKE_ASM_COMPILER arm-none-eabi-gcc)

# Set the CPU and FPU options
set(CPU_FLAGS "-mcpu=cortex-m4 -march=armv7e-m -mthumb")
set(FPU_FLAGS "-mfpu=fpv4-sp-d16 -mfloat-abi=hard")

# Optimization and debugging flags
set(OPTIMIZATION_FLAGS "-Os -ffunction-sections -fdata-sections")
set(DEBUG_FLAGS "-g -gstrict-dwarf -MD")
set(WARNING_FLAGS "-Wall -Wextra -pedantic")

# Set the compiler flags
set(CMAKE_C_FLAGS "${CPU_FLAGS} ${FPU_FLAGS} ${OPTIMIZATION_FLAGS} ${DEBUG_FLAGS} ${WARNING_FLAGS} -std=c99 -w -D__MSP432E401Y__ -Dgcc")
set(CMAKE_ASM_FLAGS "${CMAKE_C_FLAGS}")

# Set the linker script and linker flags
set(LINKER_SCRIPT ${CMAKE_SOURCE_DIR}/msp432e401y.lds)
set(CMAKE_EXE_LINKER_FLAGS "${CPU_FLAGS} ${FPU_FLAGS} -Wl,--gc-sections -T ${LINKER_SCRIPT} -static -nostartfiles -specs=nano.specs -specs=nosys.specs")

# Include directories
include_directories(
    ${CMAKE_SOURCE_DIR}/sw/inc
)

# Define the variable for source files (add your own source files here)
set(SRC_FILES
    src/startup_msp432e401y_gcc.c
    src/system_msp432e401y.c
    src/main.c
)

# Add the executable using the SRC_FILES variable
add_executable(exe.elf ${SRC_FILES})

# Convert the ELF to BIN
add_custom_command(TARGET exe.elf POST_BUILD
    COMMAND arm-none-eabi-objcopy -O binary $<TARGET_FILE:exe.elf> ${CMAKE_BINARY_DIR}/exe.bin
    COMMENT "Converting ELF to BIN"
)

# Print the size of the binary
add_custom_command(TARGET exe.elf POST_BUILD
    COMMAND arm-none-eabi-size $<TARGET_FILE:exe.elf>
    COMMENT "Displaying binary size"
)

# Clean up object and dependency files after build
add_custom_command(TARGET exe.elf POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E remove ${CMAKE_BINARY_DIR}/*.o ${CMAKE_BINARY_DIR}/*.d
    COMMENT "Removing object and dependency files"
)

