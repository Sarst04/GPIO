# GPIO (General-Purpose Input/Output Controller)

A parametrizable, memory-mapped General-Purpose Input/Output (GPIO) controller implemented in SystemVerilog.

The GPIO controller provides configurable input and output interfaces, synchronized input sampling, programmable edge-detection interrupts, and a lightweight memory-mapped register interface.

## Features

- **Parametrizable I/O Width:** Configurable number of GPIO input (`GPIO_IN_NUM`) and output (`GPIO_OUT_NUM`) signals.
- **Synchronized Input Sampling:** Two-stage input sampling registers to synchronize external GPIO inputs to the system clock domain.
- **Edge Detection:** Detects input transitions by comparing consecutive synchronized samples.
- **Programmable Interrupt Control:** Independently enables or disables edge-detection interrupts for each input through the control register.
- **Interrupt Status:** Maintains pending interrupt status bits until explicitly cleared by software.
- **Memory-Mapped Interface:** Provides a simple 32-bit read/write register interface for peripheral access.
- **Asynchronous Reset:** Initializes output signals, interrupt configuration, and status registers.

## Architecture

The GPIO controller consists of four main functional blocks:

1. **Input Synchronization:** Samples external GPIO inputs through two sequential registers.
2. **Edge Detection:** Detects transitions by comparing the current and previous synchronized input samples using XOR logic.
3. **Interrupt Generation:** Applies the programmable control mask to detected transitions and records pending events in the status register.
4. **Register Interface:** Provides software access to GPIO inputs, outputs, interrupt status, and interrupt configuration.

The interrupt output is asserted whenever at least one enabled interrupt event is pending.

## Synthesis

This core is designed for synthesis using standard SystemVerilog-compatible tools.

| Tool | Purpose |
|---|---|
| Slang | SystemVerilog linting and elaboration |
| Yosys/Slang | Logic synthesis |

## Parameters

| Parameter | Type | Default | Description |
|---|---|---:|---|
| `GPIO_IN_NUM` | `int` | `8` | Number of GPIO input signals. |
| `GPIO_OUT_NUM` | `int` | `8` | Number of GPIO output signals. |

## Memory Map

The GPIO controller uses a 32-bit memory-mapped register interface. Register addresses are decoded using `address[3:0]`.

| Offset | Register | Access | Description |
|---|---|---|---|
| `0x0` | `GPIO_OUT` | R/W | GPIO output data register. |
| `0x4` | `GPIO_IN` | R/O | Synchronized GPIO input values. |
| `0x8` | `STATUS` | R/W | Pending edge-detection interrupt status. Writing to this register clears all pending status bits. |
| `0xC` | `CONTROL` | R/W | Per-input interrupt enable mask. |

### Register Details

**GPIO_OUT (0x0)**

- Read: Returns the current output register value.
- Write: Updates the GPIO output signals.
- Only the least significant `GPIO_OUT_NUM` bits are implemented.

**GPIO_IN (0x4)**

- Read: Returns the synchronized GPIO input values.
- The input signals are sampled through two sequential registers.
- Only the least significant `GPIO_IN_NUM` bits are implemented.

**STATUS (0x8)**

- Read: Returns the pending interrupt status bits.
- Write: Clears all pending interrupt status bits.
- Status bits are set when an input transition is detected and the corresponding control bit is enabled.

**CONTROL (0xC)**

- Read: Returns the interrupt enable configuration.
- Write: Configures interrupt detection independently for each GPIO input.
- A control bit of `1` enables interrupt detection for its corresponding input.

## Interrupt Generation

The GPIO controller detects both rising and falling transitions on synchronized input signals.

For each input bit:

`edge_detected = gpioInReg ^ gpioInRegPre`

The pending interrupt status is updated as follows:

`statusReg_next = statusReg | (edge_detected & controlReg)`

The global interrupt output is generated using:

`irq = |(statusReg & controlReg)`

Pending interrupt events remain recorded until software writes to the `STATUS` register.

## Ports

| Port Name | Direction | Width | Description |
|---|---|---|---|
| `clk` | Input | 1 | System clock. |
| `rst` | Input | 1 | Asynchronous active-high reset. |
| `GPIO_In` | Input | `GPIO_IN_NUM` | External GPIO input signals. |
| `GPIO_Out` | Output | `GPIO_OUT_NUM` | GPIO output signals. |
| `readRequest` | Input | 1 | Asserted during a valid register read transaction. |
| `writeRequest` | Input | 1 | Asserted during a valid register write transaction. |
| `address` | Input | 32 | Memory-mapped register address. |
| `dataIn` | Input | 32 | Write data bus. |
| `dataOut` | Output | 32 | Read data bus. |
| `irq` | Output | 1 | Combined GPIO interrupt request. |

## Integration

The GPIO controller is intended to be integrated as a memory-mapped peripheral within a RISC-V SoC.

It can be connected to a processor data bus or an interconnect through its read/write request signals, address bus, and data interface.

The `irq` output can be connected to an interrupt aggregation unit such as a RISC-V Platform-Level Interrupt Controller (PLIC).

## License

This project is licensed under the [CERN Open Hardware Licence Version 2 - Strongly Reciprocal (CERN-OHL-S-2.0)](LICENSE).