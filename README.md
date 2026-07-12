UART RECEIVER (VERILOG)
## Project Features
* **Double-Flop Synchronization**: Two stages of flip-flops (`rx_meta` and `rx_sync`) are used to synchronize the external asynchronous `serial_in` to the local clock domain, ensuring system stability.
* **Middle-Bit Sampling**: Samples incoming data bits precisely at the center of each bit period (`CLKS_PER_BIT >> 1` for start bit, and `CLKS_PER_BIT - 1` for subsequent data bits) to maximize noise margin.
* **LSB-First Data Accumulation**: Accumulates bits in standard UART Least Significant Bit (LSB) first order into an 8-bit output register (`data_buf`).
* **Active-Low Asynchronous Reset**: Restores FSM to its safe default `IDLE` state instantly.

##  Architecture & State Machine

The receiver uses a 4-state Finite State Machine (FSM):
1. **`IDLE` (2'b00)**: The receiver waits with the clock counter and bit index reset. It continuously monitors `rx_sync`. When `rx_sync` drops to `0` (detecting a Start Bit edge), it transitions to the `START` state.
2. **`START` (2'b01)**: The FSM waits until the clock counter reaches the exact middle of the start bit duration (108 clock cycles). If the line is still low, it confirms a valid start bit and transitions to `RECV`. If it was a noise glitch (line went back to high), it returns to `IDLE`.
3. **`RECV` (2'b12)**: The receiver waits for a full bit duration (217 cycles) to reach the center of each data bit, sampling the input directly into `data_buf[bit_index]`. Once all 8 bits are collected (`bit_index == 7`), it moves to `STOP`.
4. **`STOP` (2'b11)**: The receiver waits for the duration of the stop bit, updates the final `data_out` register, asserts `data_ready` high for exactly one clock cycle, and returns to `IDLE`.

## System Specifications & Timing
For this simulation, the testbench is configured with the following parameters:
* **System Clock Frequency**: 25 MHz (Clock Period = 40 ns)
* **Baud Rate**: 115,200 bps (standard high-speed communication)
* **Clock Cycles Per Bit (`CLKS_PER_BIT`)**: 217 cycles 
  
## Simulation and Testbench

The testbench (`uart_tb.v`) stimulates the module by sending two complete test frames back-to-back:
1. **Frame 1**: `8'h3C` (`8'b00111100`) — Sent LSB-first (`0 -> 0 -> 1 -> 1 -> 1 -> 1 -> 0 -> 0`)
2. **Frame 2**: `8'h2F` (`8'b00101111`) — Sent LSB-first (`1 -> 1 -> 1 -> 1 -> 0 -> 1 -> 0 -> 0`)

### Verification Log Output
When compiled and executed successfully, the simulation outputs:
RX[1] @ <time> ns = 3c
RX[2] @ <time> ns = 2f
Final data_out = 2f
PASS: TWO BYTES RECEIVED SUCCESSFULLY
