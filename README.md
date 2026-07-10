# UART Transceiver in Verilog

## Overview

This project implements a complete **Universal Asynchronous Receiver/Transmitter (UART)** in Verilog HDL. The design includes independent **UART Transmitter (TX)** and **UART Receiver (RX)** modules along with a configurable **Baud Rate Generator**, enabling reliable full-duplex serial communication between digital systems.

The UART follows the standard asynchronous serial protocol using:

* 1 Start Bit
* 8 Data Bits
* No Parity
* 1 Stop Bit (8N1)

The design is fully modular, making it suitable for FPGA-based embedded systems and larger SoC designs.

---

## Features

* UART Transmitter (TX)
* UART Receiver (RX)
* Full-duplex communication
* Configurable system clock frequency
* Configurable baud-rate generation
* Standard UART 8N1 protocol
* Independent TX and RX enable pulses
* FSM-based implementation
* Synthesizable Verilog HDL
* Suitable for FPGA implementation

---

## Project Structure

```text
UART-Verilog/
│
├── README.md
├── LICENSE
│
├── rtl/
│   ├── uart_tx.v
│   ├── uart_rx.v
│   ├── Baud_Rate.v
│   └── uart_top.v
│
├── sim/
│   ├── tb_uart_tx.v
│   ├── tb_uart_rx.v
│   ├── tb_uart_top.v
│   └── waveform.png
│
├── docs/
│   ├── UART_Frame.png
│   ├── TX_FSM.png
│   ├── RX_FSM.png
│   └── Block_Diagram.png
│
└── constraints/
    └── Basys3.xdc
```

---

# UART Architecture

```text
                  +----------------------+
                  | Baud Rate Generator  |
                  +----------+-----------+
                             |
                 +-----------+-----------+
                 |                       |
            TX Baud Tick           RX Oversampling Tick
                 |                       |
         +-------v------+        +-------v------+
         | UART TX FSM  |        | UART RX FSM  |
         +-------+------+        +-------+------+
                 |                       |
              TX Output              RX Input
                 |                       |
                 +-----------+-----------+
                             |
                      UART Interface
```

---

# UART Frame Format

Each transmitted byte follows the standard **8N1** frame.

```text
Idle  Start   D0 D1 D2 D3 D4 D5 D6 D7   Stop
 1      0      LSB -------------> MSB     1
```

| Field  | Bits |
| ------ | ---- |
| Start  | 1    |
| Data   | 8    |
| Parity | None |
| Stop   | 1    |

---

# UART Transmitter

The transmitter converts an 8-bit parallel word into a serial bit stream.

## Operation

1. Waits in the Idle state.
2. Loads input data when `tx_valid` is asserted.
3. Sends the Start bit.
4. Transmits all 8 data bits (LSB first).
5. Sends the Stop bit.
6. Returns to Idle and asserts `tx_done`.

### Interface

| Signal      | Direction | Description                           |
| ----------- | --------- | ------------------------------------- |
| `data_in`   | Input     | Parallel data to transmit             |
| `tx_valid`  | Input     | Begins transmission                   |
| `baud_tick` | Input     | Baud-rate enable pulse                |
| `tx`        | Output    | UART serial output                    |
| `tx_busy`   | Output    | Transmission in progress              |
| `tx_ready`  | Output    | Ready to accept new data              |
| `tx_done`   | Output    | One-clock transmission complete pulse |

### TX State Machine

```text
Idle
 │
 ▼
Start
 │
 ▼
Data (8 bits)
 │
 ▼
Stop
 │
 ▼
Idle
```

---

# UART Receiver

The receiver reconstructs serial data received on the RX line into an 8-bit parallel word.

To improve sampling accuracy, the receiver uses **16× oversampling**. After detecting the falling edge of the Start bit, it samples near the center of each bit period before shifting the received bit into an internal register.

## Operation

1. Monitor RX line while idle.
2. Detect Start bit.
3. Confirm the Start bit at the middle of the bit period.
4. Sample each of the 8 data bits.
5. Verify the Stop bit.
6. Assert `rx_done` and present the received byte on `data_out`.

### Receiver Features

* Start-bit detection
* Mid-bit sampling
* 16× oversampling
* Data-valid indication
* Stop-bit verification

### RX State Machine

```text
Idle
 │
 ▼
Start Detect
 │
 ▼
Receive Data
 │
 ▼
Stop Bit
 │
 ▼
Data Valid
 │
 ▼
Idle
```

---

# Baud Rate Generator

The baud-rate generator produces timing pulses for both transmission and reception.

The transmitter requires one enable pulse per transmitted bit, while the receiver operates using a **16× oversampling clock** to improve robustness against timing differences between communicating devices.

For a 100 MHz system clock and a baud rate of 9600 bps:

* TX Tick = 9600 Hz
* RX Tick = 153600 Hz (16×)

The clock frequency can be changed by modifying the `clk_speed` parameter.

---

# Simulation

Simulation verifies:

* UART frame generation
* Correct bit ordering (LSB first)
* Baud timing
* Start and Stop bits
* Successful reception
* Loopback communication between TX and RX

---

# Applications

* FPGA-to-PC serial communication
* Embedded debugging interfaces
* Sensor communication
* Soft-core processor peripherals
* Bootloaders
* Serial command interfaces
* FPGA development boards

---

# Future Improvements

* Configurable baud rate
* Configurable data width
* Even/Odd parity support
* Two stop-bit mode
* FIFO buffering
* AXI/APB UART peripheral
* Interrupt generation
* Hardware flow control (RTS/CTS)
* Error detection (Framing, Parity, Overrun)
* DMA interface

---

# Author

**Sai Shubham Biswal**

A modular UART implementation in Verilog HDL designed for FPGA-based digital systems and serial communication applications.

