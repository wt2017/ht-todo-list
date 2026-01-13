The service-board/app directory contains six embedded firmware projects for various microcontroller-based expansion boards used in industrial control and measurement systems:

1. **app_gd32f130f8** – Wireless module IO control board for electric energy acquisition devices. Controls five GPIOs (reset, power, LEDs) via libcmx with register‑based configuration and a semantic command interface.

2. **APP_GD32F305** – PCAN (CAN bus) interface board using GD32F305. Provides CAN expansion via USB, supports configurable baud rates, and enables CAN frame transmission/reception for the CME‑G21H extension board.

3. **app_gd32h757** – Signal analysis and acquisition application for GD32H757. Implements I2C slave protocol, USB CDC ACM, AD7606 ADC continuous sampling (up to 20 kHz), signal calibration, frequency/phase analysis, and EEPROM configuration storage.

4. **APP_STM32F103** – Digital I/O (switch量) board program for STM32F103/GD32F103. Supports multiple extension boards (CME‑G20H to G24H) and communicates with the host via I2C.

5. **APP_STM32F407** – Analog acquisition board based on STM32F407ZET6. Features AD7606 ADC sampling (configurable up to 200 kHz), I2C communication, and output control. Derived from a development board example.

6. **gd32f305_pcan** – USB CDC ACM demo firmware for GD32F4xx, implementing a virtual serial port over USB for data transfer and debugging.

These projects collectively provide hardware abstraction and peripheral drivers for expansion boards used in the service‑board ecosystem, covering digital I/O, analog acquisition, CAN communication, wireless module control, and signal processing.
