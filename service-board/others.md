The service-board directory contains the following projects beyond app and boot:

1. **GCP_extend_mod** - NOT USED – General Communication Platform extension module. A cross‑platform communication middleware with pre‑built binaries for multiple architectures (ARM, x86, various Linux distributions). Includes scripts for deployment, SSL/TLS support, and protocol handling (Class C, OpenSSL). Used for device‑to‑cloud or inter‑device communication in HangTu's embedded systems.

2. **libcmx** – Library for controlling modular I/O expansion boards (DI/DO/AI). Provides a dynamic library (`libcmx.so`) and command‑line tool (`demo‑libcmx`) to scan, read, and write digital/analog I/O via I2C. Includes examples for Raspberry Pi testing and firmware upgrade utilities.

3. **libcmx_demo** – Demonstration code and comprehensive documentation for the libcmx library. Contains example programs for DI, DO, AI, SOE (sequence of events), and onboard DO (front‑panel LEDs, relays). Shows how to initialize the library, read/write channels, enable SOE recording, and control motherboard GPIOs.

4. **STM32_AD7606** - NOT USED – Analog acquisition board firmware for STM32F407ZET6 with AD7606 ADC. Supports configurable sampling rates (up to 200 kHz), Modbus protocol, and USB communication. Derived from a development board example.

5. **STM32_Modbus** - NOT USED – Modbus protocol implementation for STM32 microcontrollers. Provides relay control and communication for DC802 environment. Built with Keil5.14.0.0.

6. **STM32F0_USB_HID_SPI_BRIDGE** – USB HID‑SPI bridge firmware for STM32F072. Implements a USB HID device that acts as a bridge between host and SPI peripherals. Includes PC test tools (PortHelper) and demonstrates USB HID report sending/receiving.

7. **Test_STM32_I2C_Slave** – Host‑side test code for STM32 I2C slave projects. Includes Raspberry Pi utilities (i2c‑tools, libsoc_i2c) and a Go‑based firmware upgrade tool (`pivoyager`). Used to validate I2C communication with STM32F103/F407 slave boards.

8. **UPS** – Uninterruptible power supply firmware for STM32F072. Integrates EasyFlash for parameter storage and SFUD for SPI flash management. Likely monitors power status and manages battery backup.

These projects collectively form the software ecosystem for HangTu's service‑board platform, covering communication middleware, I/O control libraries, protocol stacks, firmware for various expansion boards, and testing utilities.
