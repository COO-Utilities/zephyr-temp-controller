# AD7124 Temperature Controller

A Zephyr application for reading temperature from the AD7124 24-bit ADC's on-chip temperature sensor.

## Overview

This application demonstrates SPI communication with the Analog Devices AD7124 ADC, configured to continuously read from its internal temperature sensor. The temperature readings are logged to the console every 3 seconds.

## Hardware Requirements

- **Board**: STM32 Nucleo-H563ZI (or compatible STM32 board)
- **ADC**: AD7124 (or AD7124-4/AD7124-8)
- **Connections**:
  - SPI1_SCK: PA5
  - SPI1_MISO: PG9
  - SPI1_MOSI: PB5
  - SPI1_CS: PD14 (GPIO, active low)

## Features

- **SPI Mode 3** (CPOL=1, CPHA=1) communication
- **Continuous conversion** mode
- On-chip temperature sensor reading (±1°C typical accuracy)
- Automatic ADC configuration:
  - Bipolar mode
  - Internal 2.5V reference with buffers enabled
  - SINC3 digital filter
  - PGA gain = 1
- Temperature conversion formula calibrated for AD7124 internal sensor
- Robust error handling and status logging

## Building

### Prerequisites

1. [Zephyr SDK](https://docs.zephyrproject.org/latest/develop/getting_started/index.html) installed
2. Zephyr workspace set up
3. West tool configured

### Build Commands

```bash
# From the project directory
west build -b nucleo_h563zi

# Clean build
west build -b nucleo_h563zi --pristine

# Flash to board
west flash
```

## Configuration

### Key Configuration Options ([prj.conf](prj.conf))

- `CONFIG_SPI=y` - Enable SPI driver
- `CONFIG_ADC=y` - Enable ADC subsystem
- `CONFIG_ADC_AD7124=y` - Enable AD7124 driver
- `CONFIG_CPP=y` - Enable C++ support
- `CONFIG_LOG=y` - Enable logging
- `CONFIG_CBPRINTF_FP_SUPPORT=y` - Enable floating-point printf support

### Device Tree

The board-specific configuration is in [boards/nucleo_h563zi.overlay](boards/nucleo_h563zi.overlay), which defines:
- SPI1 pin mappings
- Chip select GPIO
- AD7124 device node with SPI mode settings

## Usage

After flashing, connect a serial terminal to the board's USB virtual COM port (115200 baud). You should see output similar to:

```
[00:00:00.000,000] <inf> ad7124: AD7124 on-chip temperature controller
[00:00:00.003,000] <inf> ad7124: CFG0=0x0fe0 CH0=0x8211 ADC_CTRL=0x0100
[00:00:00.003,000] <inf> ad7124: Waiting for first ready...
[00:00:00.010,000] <inf> ad7124: DATA=0x800d42  Temp≈25.32 °C
[00:00:03.010,000] <inf> ad7124: DATA=0x800d41  Temp≈25.31 °C
[00:00:06.010,000] <inf> ad7124: DATA=0x800d43  Temp≈25.33 °C
```

## Code Structure

- [src/main.cpp](src/main.cpp) - Main application code
  - SPI read/write helpers
  - AD7124 register definitions
  - Temperature sensor configuration
  - Continuous reading loop
  - Temperature conversion algorithm

## Temperature Conversion

The internal temperature sensor outputs are converted using:

```cpp
temp_celsius = (code - 8388608) / 13584.0 - 272.5
```

Where `code` is the 24-bit offset binary ADC result. This formula is based on the AD7124 datasheet's internal temperature sensor specifications.

## Customization

### Changing Sample Rate

Modify the `REG_FILTER0` value in `config_temp()`:

```cpp
wr24(REG_FILTER0, 0x060180);  // Current setting
// Lower values = faster sampling, higher noise
// Higher values = slower sampling, lower noise
```

### Changing Reading Interval

Adjust the sleep time in the main loop:

```cpp
k_msleep(3000);  // Current: 3 seconds
```

### Using External Inputs

To measure external analog inputs instead of temperature, modify the `config_temp()` function to configure different channel inputs. Refer to the AD7124 datasheet for analog input pin assignments.


## References

- [AD7124 Datasheet](https://www.analog.com/en/products/ad7124.html)
- [Zephyr SPI API Documentation](https://docs.zephyrproject.org/latest/hardware/peripherals/spi.html)
- [STM32H563ZI Reference Manual](https://www.st.com/en/microcontrollers-microprocessors/stm32h563zi.html)
