The service-board/boot directory contains three bootloader/I2C slave projects:

1. **GD32F103_I2C_BOOT** – A bootloader for GD32F103/STM32F103 microcontrollers on CME‑G20H/G21H/G22H extension boards. Implements I2C‑based in‑application programming (IAP) with the following features:
   - Flash partitioning: 64 KB bootloader, 128 KB application, 4 KB upgrade flag.
   - Supports online firmware upgrade via libcmx (`-U` command) with CRC8‑checked bin headers.
   - Backup/restore capability to/from EEPROM (56 KB backup region).
   - Includes Python script for adding header (magic, size, timestamp, git version, CRC8) to bin files.
   - Defines structured command set (erase, read, write, exit, check, EEPROM operations) and error codes.

2. **STM32F103_I2C_Slave** – I2C slave demonstration for STM32F103 using STM32CubeIDE and HAL library. Focuses on continuous I2C slave communication using `HAL_I2C_Slave_Seq_Receive_IT` and `HAL_I2C_Slave_Seq_Transmit_IT` for reliable sequential data transfer. Serves as a reference for implementing I2C slave endpoints in expansion boards.

3. **STM32407_I2C_Slave** – Similar I2C slave demonstration for STM32F407, also built with STM32CubeIDE/HAL. Provides the same continuous I2C slave functionality, likely used for analog‑board communication or as a generic I2C slave template.

These boot‑related projects enable firmware updates and robust I2C slave communication for the service‑board ecosystem, supporting both upgrade workflows and host‑peripheral data exchange.
