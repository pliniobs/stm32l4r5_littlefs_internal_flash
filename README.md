## STM32L4R5 LittleFS Internal Flash

A CMake-based STM32 embedded project demonstrating LittleFS filesystem integration on the STM32L4R5 microcontroller using internal flash memory for persistent storage.

## Project Overview

This project implements a lightweight filesystem (LittleFS) on the STM32L4R5 microcontroller, utilizing the internal flash memory (Bank 2) for file storage. The application tracks boot counts in a persistent file, demonstrating basic filesystem operations like mounting, file creation, reading, and writing.

### Key Features

- **LittleFS Integration**: Embedded filesystem with wear-leveling support
- **Internal Flash Storage**: Uses Bank 2 of STM32L4R5 (1 MB, starting at 0x08100000)
- **CMake Build System**: Modern build configuration with support for multiple toolchains
- **HAL-based Flash Operations**: Direct flash read/write/erase operations via STM32 HAL
- **Dual Toolchain Support**: Compatible with both GCC Arm and starm-clang compilers

## Hardware Configuration

### Target Device
- **MCU**: STM32L4R5ZITx
- **Flash Memory for LittleFS**: Bank 2
  - Base Address: `0x08100000`
  - Size: 256 pages × 4 KB = 1 MB total
  - Page Size: 4096 bytes

### System Clock
- **Oscillator**: MSI (Multi-Speed Internal)
- **PLL**: Enabled with 60× multiplier
- **Core Frequency**: 120 MHz
- **Latency**: 5 wait states

## Project Structure

```
├── Core/
│   ├── Inc/        # Application headers
│   └── Src/        # Application sources (main.c)
├── Drivers/
│   ├── STM32L4xx_HAL_Driver/  # Hardware Abstraction Layer
│   └── CMSIS/                 # ARM Cortex-M4 definitions
├── Middlewares/
│   └── littlefs/   # LittleFS filesystem library
├── cmake/
│   ├── gcc-arm-none-eabi.cmake     # GCC toolchain config
│   ├── starm-clang.cmake            # STARM-Clang toolchain config
│   └── stm32cubemx/                 # STM32CubeMX generated CMake
├── CMakeLists.txt  # Main CMake configuration
├── CMakePresets.json
└── stm32l4r5_littlefs_internal_flash.ioc  # STM32CubeMX project file
```

## LittleFS Configuration

The filesystem is configured in main.c with the following parameters:

```c
.read_size = 64        // Read operation block size
.prog_size = 64        // Program operation block size
.block_size = 4096     // Flash page size
.block_count = 256     // Number of pages
.cache_size = 256      // Cache buffer size
.lookahead_size = 32   // Lookahead buffer for free block tracking
.block_cycles = 500    // Write cycles before wear-leveling activation
```

## Flash Block Device Operations

### `flash_read_mem()`
Reads data from flash memory in 32-bit chunks.
- **Parameters**: Block number, offset, buffer, size
- **Returns**: `LFS_ERR_OK` on success

### `flash_prog_mem()`
Programs (writes) data to flash using 64-bit double-word operations.
- **Parameters**: Block number, offset, data buffer, size
- **Flash State**: Unlocked during write, locked after completion
- **Returns**: `LFS_ERR_OK` on success, `LFS_ERR_IO` on failure

### `flash_erase_mem()`
Erases a flash page in Bank 2.
- **Parameters**: Block number (used as page index)
- **Bank**: FLASH_BANK_2
- **Clears**: OPTVERR flag before erase
- **Returns**: `LFS_ERR_OK` on success, `LFS_ERR_IO` on failure

### `flash_sync_mem()`
Synchronization operation (placeholder - returns 0).

## Building the Project

### Prerequisites

- **CMake** ≥ 3.22
- **Build System**: Ninja or Make
- **Toolchain**: Either GCC Arm Embedded or STARM-Clang
  - GCC: `arm-none-eabi-*` tools in PATH
  - STARM-Clang: `starm-*` tools in PATH

### Build with CMake Presets

```bash
# Configure and build with default preset
cmake --preset default
cmake --build --preset default

# Or with debug configuration
cmake --preset debug
cmake --build --preset debug
```

### Build Output

The compiled executable generates:
- `stm32l4r5_littlefs_internal_flash.elf` - ELF binary
- `stm32l4r5_littlefs_internal_flash.map` - Linker map file
- `compile_commands.json` - Compilation database for IDE integration

## Flashing & Running

1. **Connect** the STM32L4R5 via ST-Link debug interface
2. **Flash** using STM32CubeProgrammer or OpenOCD:
   ```bash
   st-flash write build/Debug/stm32l4r5_littlefs_internal_flash.elf
   ```
3. **Monitor** the debug output via serial console (if UART configured)

## Application Behavior

The application performs the following on startup:

1. Initializes the MCU and configures system clock to 120 MHz
2. Unlocks flash memory and clears error flags
3. Mounts the LittleFS filesystem (formats on first boot if needed)
4. Opens or creates a `boot_count` file
5. Reads the current boot count value
6. Increments and writes the updated boot count back to flash
7. Closes the file and unmounts the filesystem
8. Enters an infinite loop

## Development Environment

### IDE Support

The project is optimized for **STM32CubeIDE** with CMake support:

- **settings.json**: VS Code configuration for Clangd integration
- **.clangd**: Language server configuration for code completion
- **compile_commands.json**: Generated compilation database

### Code Organization

The main application logic is located in main.c, which includes:
- System initialization via `SystemClock_Config()`
- LittleFS configuration structure
- Flash block device abstraction functions
- Boot count persistence example

## Troubleshooting

### Common Issues

| Issue | Solution |
|-------|----------|
| "Filesystem invalid" on mount | Check `LITTLEFS_FLASH_BASE_ADDRESS` alignment and `FLASH_PAGE_SIZE` configuration |
| Flash program errors | Ensure flash is unlocked before operations; verify `FLASH_BANK_2` is selected in erase operations |
| Boot count not persisting | Verify file is closed properly with `lfs_file_close()` before unmounting |

## References

- [LittleFS Documentation](https://github.com/littlefs-project/littlefs)
- [STM32L4R5 Reference Manual](https://www.st.com/resource/en/reference_manual/dm00083793-stm32l476xx-advanced-armbased-32bit-mcu-reference-manual-stmicroelectronics.pdf)
- [STM32CubeL4 Firmware Package](https://github.com/STMicroelectronics/STM32CubeL4)

## License

This project uses:
- **STM32CubeL4**: ST Proprietary License
- **LittleFS**: BSD-3-Clause License
- **Application Code**: Custom implementation

---
