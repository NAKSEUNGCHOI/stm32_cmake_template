Here's your complete **documentation** for setting up, building, flashing, and debugging your STM32F411RE **blinky LED** project using **CMake & Ninja** on **Linux**. 

I'll structure it so you can easily follow or reproduce it in the future.

---

# **STM32F411RE Blinky LED (CMake & Ninja on Linux)**

## **📌 Overview**
This guide documents the steps to:
- **Set up** the STM32F411RE development environment on **Linux**
- **Build** firmware using **CMake & Ninja**
- **Flash & Debug** using **OpenOCD & GDB**
- **Verify the working LED blink project**

---

## **1️⃣ Prerequisites**
### **Install Required Tools**
Ensure you have the following installed:
```bash
sudo apt update && sudo apt install -y \
    cmake ninja-build \
    gcc-arm-none-eabi \
    gdb-multiarch \
    openocd
```
- `cmake`: CMake for project generation
- `ninja-build`: Fast build system
- `gcc-arm-none-eabi`: ARM GCC compiler toolchain
- `gdb-multiarch`: GDB for debugging embedded systems
- `openocd`: OpenOCD for flashing & debugging

---

## **2️⃣ Project Structure**
Your project should be organized as follows:
```
test_project/
├── Core/
│   ├── Inc/
│   │   ├── main.h
│   │   ├── stm32_assert.h
│   │   ├── stm32f4xx_it.h
│   ├── Src/
│   │   ├── main.c
│   │   ├── stm32f4xx_it.c
│   │   ├── syscalls.c
│   │   ├── sysmem.c
│   │   ├── system_stm32f4xx.c
│   ├── Startup/
│   │   ├── startup_stm32f411retx.s
├── Drivers/
│   ├── CMSIS/
│   │   ├── Core/Include
│   │   ├── Device/ST/STM32F4xx/Include
│   ├── STM32F4xx_HAL_Driver/
│   │   ├── Inc/
│   │   ├── Src/
├── Debug/
│   ├── STM32F411RETX_FLASH.ld  # Linker script
├── CMakeLists.txt
└── build/  # CMake build directory
```
---

## **3️⃣ CMakeLists.txt**
Your `CMakeLists.txt` file should be as follows:

```cmake
cmake_minimum_required(VERSION 3.16)

# Project name
project(blink C ASM)

# Define CPU, FPU, and Float ABI
set(CPU_FLAGS "-mcpu=cortex-m4 -mthumb -mfpu=fpv4-sp-d16 -mfloat-abi=softfp")

# Compiler flags
set(COMMON_FLAGS "${CPU_FLAGS} -Wall -ffunction-sections -fdata-sections -g -O2")

# Set toolchain
set(CMAKE_SYSTEM_NAME Generic)
set(CMAKE_SYSTEM_PROCESSOR arm)
set(CMAKE_C_COMPILER arm-none-eabi-gcc)
set(CMAKE_ASM_COMPILER arm-none-eabi-gcc)

# Linker flags
set(CMAKE_EXE_LINKER_FLAGS "${COMMON_FLAGS} -Wl,--gc-sections --specs=nosys.specs -T${CMAKE_SOURCE_DIR}/Debug/STM32F411RETX_FLASH.ld")

# Define macros
add_compile_definitions(USE_FULL_LL_DRIVER)
add_compile_definitions(USE_FULL_ASSERT)

# Include directories
include_directories(
    Core/Inc
    Drivers/CMSIS/Core/Include
    Drivers/CMSIS/Device/ST/STM32F4xx/Include
    Drivers/STM32F4xx_HAL_Driver/Inc
)

# Source files
set(SRCS
    Core/Src/main.c
    Core/Src/stm32f4xx_it.c
    Core/Src/syscalls.c
    Core/Src/sysmem.c
    Core/Src/system_stm32f4xx.c
    Core/Startup/startup_stm32f411retx.s
    Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_ll_gpio.c
    Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_ll_exti.c
    Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_ll_usart.c
    Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_ll_rcc.c
    Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_ll_utils.c
    Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_ll_pwr.c
)

# Define executable
add_executable(blink.elf ${SRCS})

# Linker options
target_link_options(blink.elf PRIVATE
    "-T${CMAKE_SOURCE_DIR}/Debug/STM32F411RETX_FLASH.ld"
    "-Wl,-Map=blink.map"
)

# Compiler options
target_compile_options(blink.elf PRIVATE ${COMMON_FLAGS})

# Add necessary definitions
target_compile_definitions(blink.elf PRIVATE USE_FULL_LL_DRIVER)
```

---

## **4️⃣ Building the Project**
After setting up the files, compile the project:

```bash
mkdir -p build && cd build
cmake -G Ninja ..
ninja
```

If everything is correct, you'll see:
```
[100%] Built target blink.elf
```

---

## **5️⃣ Flashing the Firmware**
### **Using OpenOCD & GDB**
#### **Start OpenOCD**
```bash
openocd -f interface/stlink.cfg -f target/stm32f4x.cfg
```
Leave this terminal open.

#### **Connect GDB**
Open a new terminal:
```bash
cd ~/workspace/test_project/build
arm-none-eabi-gdb blink.elf
```
Inside GDB:
```gdb
target remote :3333
monitor reset halt
load
monitor reset init
continue
```

#### **Exit GDB**
```gdb
(gdb) detach
(gdb) quit
```
Then stop OpenOCD with `Ctrl + C`.

---

## **6️⃣ Flash Without Debugging**
If you just want to flash without debugging:
```bash
openocd -f interface/stlink.cfg -f target/stm32f4x.cfg -c "program blink.elf verify reset exit"
```
---

## **7️⃣ Verify the Flash**
To check if the firmware is correctly written:
```gdb
(gdb) monitor mdw 0x08000000 10
```
It should display the firmware in flash.

---

## **8️⃣ Troubleshooting**
| **Issue**                           | **Solution**                                         |
|--------------------------------------|------------------------------------------------------|
| `arm-none-eabi-gcc: fatal error`     | Ensure ARM toolchain is installed (`gcc-arm-none-eabi`) |
| `Undefined reference to LL_*`        | Ensure `USE_FULL_LL_DRIVER` is defined in `CMakeLists.txt` |
| `OpenOCD cannot detect ST-Link`      | Try `sudo openocd` or reconnect ST-Link |
| `HardFault on reset`                 | Check `SystemClock_Config()` implementation |

---

## **🎯 Conclusion**
You now have:
✅ A **working STM32F411RE blinky LED** project  
✅ Built with **CMake & Ninja** on **Linux**  
✅ Flashed via **OpenOCD & GDB**  

This setup will allow you to expand into **other STM32 projects** using **LL drivers** effectively.

Let me know if you need **any modifications**! 🚀
