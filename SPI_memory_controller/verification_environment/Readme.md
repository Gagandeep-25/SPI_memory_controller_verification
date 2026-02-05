# SPI Memory Controller – Simulation Waveform Analysis

This document describes the **simulation waveform output** of the SPI Memory Controller design and its UVM-based verification environment. The waveform demonstrates correct **read/write transactions**, **handshake signaling**, **SPI serial behavior**, and **memory data integrity** across multiple operations.

---

## 📊 Overview of the Waveform

The waveform captures:
- Multiple **write and read transactions**
- Correct **SPI protocol sequencing**
- Proper **chip select (CS)** behavior
- Handshake using **ready**, **done**, and **op_done**
- Data consistency between written and read values
- Stable behavior during **reset and idle states**

---

## ⏱ Clock and Reset Signals

### `clk`
- Free-running system clock
- Drives both the SPI controller and memory FSMs
- All state transitions and serial shifting are synchronous to this clock

### `rst`
- Asserted at the beginning of simulation
- Forces the DUT into a known **idle state**
- Clears counters, FSM states, control signals, and internal registers
- Deasserted before normal SPI transactions begin

---

## 🧭 Control and Address Signals

### `wr`
- Write/read control signal
- `wr = 1` → Write operation
- `wr = 0` → Read operation

### `addr[7:0]`
- Address supplied by the driver
- Valid address range: `0–31`
- Used by the memory model to index internal storage
- Changes only at transaction boundaries

---

## 📦 Data Signals

### `din[7:0]`
- Parallel write data provided to the SPI controller
- Serialized onto `mosi` during write operations
- Changes per transaction

### `dout[7:0]`
- Parallel read data output from the SPI controller
- Captured from `miso` during read operations
- Matches previously written data, confirming correctness

---

## 🔌 SPI Interface Signals

### `cs` / `csreg` (Chip Select)
- Active-low
- Asserted (`0`) at the start of a valid SPI transaction
- Deasserted (`1`) after transaction completion
- Remains high during idle and error conditions

### `mosi` / `mosireg`
- Serial data output from SPI controller to memory
- During write:
  - Shifts `{data, address, wr}` packet bit-by-bit
- During read:
  - Sends address and control bits

### `miso` / `misoreg`
- Serial data returned from memory to SPI controller
- Active only during read transactions
- Bits captured and assembled into `dout`

---

## 🤝 Handshake and Status Signals

### `ready` / `readyreg`
- Asserted by memory after receiving read address
- Indicates memory is ready to send data
- SPI controller waits for `ready` before data capture

### `done`
- Asserted by SPI controller
- Indicates completion of current transaction
- Used by driver and monitor for synchronization

### `op_done` / `opdonereg`
- Asserted by memory
- Confirms internal memory operation completion
- Triggers SPI controller to finalize transaction

### `err`
- Asserted when an invalid address (`addr ≥ 32`) is detected
- No SPI data transfer occurs
- Transaction terminates safely

---

## 🔄 Internal Signals

### `count`
- Bit counter used for serial shifting
- Increments during MOSI/MISO transfers
- Resets at the end of each transaction

### `datain[15:0]`
- Internal shift register in memory
- Collects serial data during write or address phase

### `dataout[7:0]`
- Memory output register
- Loaded from internal memory array during read
- Shifted out on `miso`

### `state`
- FSM state of the SPI controller
- Transitions observed:
  - `idle → load → check_op → send_data / read_data → idle`
- Confirms correct protocol sequencing

---

## 🔁 Transaction Flow Observed in Waveform

### ✔ Write Transaction
1. `wr = 1`, valid `addr`, valid `din`
2. `cs` asserted low
3. Data serialized on `mosi`
4. Memory captures data
5. `op_done` asserted
6. `done` asserted
7. `cs` deasserted

### ✔ Read Transaction
1. `wr = 0`, valid `addr`
2. `cs` asserted low
3. Address serialized on `mosi`
4. Memory asserts `ready`
5. Data shifted on `miso`
6. SPI controller assembles `dout`
7. `done` asserted
8. `cs` deasserted

### ❌ Error Case
- Address out of range
- `err` asserted
- `cs` remains high
- No MOSI/MISO activity

---

## ✅ Key Observations

- Read data matches previously written values
- SPI timing and control are consistent
- Handshake signals prevent race conditions
- FSM transitions are clean and deterministic
- No spurious activity during idle
- Error handling is safe and non-intrusive

---

## 🧠 Conclusion

The waveform confirms that the **SPI memory controller and memory model function correctly**, adhering to protocol timing, ensuring data integrity, and handling error conditions gracefully. The observed transitions validate both the **RTL implementation** and the **UVM-driven stimulus**, making this design suitable for block-level verification and further extension.

---

