# Dual-Board SPI IMU Data Acquisition System

This project implements a two-board hardware architecture for acquiring data from an IMU.
The system consists of:

- **Board A**: Main controller (STM32F446R) board responsible for USB communication, power regulation, timestamping, and interfacing with the host PC.
- **Board B**: ICM-42688-P sensor board carrying an SPI IMU.
The architecture separates the processing electronics from the sensing electronics, allowing the sensor to be placed at the robot's contact point while minimizing the size and weight of the fingertip board.

# Repository Contents
- [**Images**](/exports/images) 
- [**Schematic of Board A**](/exports/Board-A-Schematic.pdf)
- [**PCB Design of Board A**](/exports/Board-A-PCB-scale.pdf)
- [**BOM of Board A**](/hardware/board-a-main/board-a-main.csv)
- [**Schematic of Board B**](/exports/Board-B-Schematic.pdf)
- [**PCB Design of Board B**](/exports/Board-B-PCB.pdf)
- [**BOM of Board B**](/hardware/board-b-sensor/board-b-sensor.csv)

# Architecture
<img width="100%" height="100%" alt="Block-diagram" src="https://github.com/user-attachments/assets/2cc2a282-ff36-4283-b88a-e46440439bed" />

Board A contains the STM32F446RE microcontroller, USB-C interface, 3.3 V regulator, debug connector, reset/boot circuitry, and the interface connector for the remote IMU.

Board B is a compact sensor board mounted near the robot gripper fingertip. It contains only the IMU and the required decoupling components to keep the sensing electronics lightweight and mechanically simple.

The two boards communicate using SPI over a 300 mm cable.

# 300 mm Cable

The cable carries:

| Signal | Purpose |
|---------|---------|
| 3.3 V | Sensor power |
| GND | Return path |
| MOSI | SPI transmit |
| GND | Additional return path |
| MISO | SPI receive |
| SCLK | SPI clock |
| CS | Chip Select |
| INT1 | Data Ready Interrupt |

Although the selected IMU supports SPI clocks up to 24 MHz, I would conservatively operate the cable at approximately 4–8 MHz without additional signal conditioning. This provides good timing margin while minimizing ringing and EMI over the cable length.

# IMU Timestamping

The IMU is configured to generate a Data Ready interrup at 300 Hz. INT1 is connected to an STM32 external interrupt (EXTI). Using a hardware timer running at 1 MHz (1 us resolution), I would expect timestamp uncertainty on the order of 2–10 us.

On every interrupt:

1. Hardware interrupt is triggered.
2. STM32 captures the current timer value.
3. SPI transaction begins immediately.
4. Sample and timestamp are stored together before transmission over USB.

# Component Selection

- **STM32F446RE**: Chosen because it has native USB support, multiple SPI peripherals, and enough processing power for reading the IMU and sending data to the host PC. It also has good documentation and development tool support.

- **ICM-42688-P**: Selected because it is a low-noise 6-axis IMU with an SPI interface and supports data rates well above the required 300 Hz. It is commonly used in robotics and motion sensing applications.

- **AP2112K-3.3**: Used to regulate the 5 V USB input to a stable 3.3 V supply for the MCU and IMU. It requires only a few external components and provides enough current for this design.

- **USBLC6-2SC6**: Added to protect the USB data lines from electrostatic discharge (ESD), improving the robustness of the USB interface.

- **USB Type-C Connector**: Provides both power and USB communication with the host PC using a standard connector.

- **JST GH 8-pin Connector**: Used for the connection between the main board and the sensor board. The locking connector is suitable for robotics applications where cables may experience vibration or movement.

# Estimated Power Budget

| Component | Current |
|-----------|---------:|
| STM32F446RE | ~35 mA |
| ICM-42688-P | ~1 mA |
| Status LED | ~2 mA |
| AP2112 Quiescent | <1 mA |
| Miscellaneous | ~2 mA |

Estimated Total: ~40–45 mA @ 3.3 V

# Tests

The first tests after receiving assembled boards would be:

1. Power Verification: Verify USB 5 and verify regulated 3.3 and check current consumption
2. MCU: Confirm SWD connectivity, program firmware, blink status LED
3. IMU Communication: Verify SPI communication and stream data to the host PC.

# Remaining Work

Given additional time, I would:

- Perform impedance calculations for the cable interface.
- Improve silkscreen documentation and assembly markings.
- Generate manufacturing outputs (Gerbers, pick-and-place, assembly drawings).
- Validate signal integrity experimentally at higher SPI clock frequencies.

# Time Spent

| Task | Time |
|------|------:|
| Architecture | 1 h |
| Schematic Design | 4 h |
| PCB Layout | 5 h |
| Component Selection | 1 h |
| Documentation | 2 h |

Total: ~13 hours

AI tools (ChatGPT) were used to:

- Research PCB layout trade-offs.
- Verify datasheet interpretation.
- Help with block diagram

All schematic capture, PCB layout, component selection, routing decisions, and trade-offs were reviewed and implemented manually.
