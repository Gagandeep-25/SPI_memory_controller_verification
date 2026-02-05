# UVM-Based SPI Memory Controller Verification

This project implements a **complete UVM verification environment** for an **SPI memory controller**, validating correct functionality across **read, write, reset, and error scenarios**. The design under test (DUT) models an SPI interface connected to an internal memory, while the verification environment ensures **protocol correctness, data integrity, and error handling**.

---

## 📌 Design Overview

The DUT consists of:
- **SPI Interface Controller (`spi_intf`)**
- **SPI Memory Model (`spi_mem`)**
- **Top-level wrapper (`top`)** connecting the controller and memory

### Key Features
- Supports SPI **read and write** operations
- Address range checking (valid: `0–31`)
- Error detection for illegal addresses (`addr > 31`)
- Handshake-based transaction completion using `done` and `op_done`
- Serial data transfer via MOSI/MISO
- Chip Select (CS) driven protocol control

---

## 🧪 Verification Architecture

The verification environment follows a **standard UVM layered architecture**:

```
test
 └── env
     ├── agent
     │   ├── sequencer
     │   ├── driver
     │   └── monitor
     └── scoreboard
```

---

## 🧩 UVM Components

### 🔹 Configuration (`spi_config`)
- Controls whether the agent operates in **active or passive** mode
- Allows easy reuse and scalability of the environment

---

### 🔹 Transaction (`transaction`)
Represents a single SPI operation and includes:
- Operation type (`read`, `write`, `reset`, `error`)
- Address and input data
- Output data, completion, and error flags
- Constrained-random address generation

#### Supported Operations
- `writed`  – Valid write
- `readd`   – Valid read
- `rstdut`  – DUT reset
- `writeerr` / `readerr` – Illegal address access

---

### 🔹 Sequences

| Sequence Name | Description |
|--------------|------------|
| `write_data` | Valid write transactions |
| `read_data`  | Valid read transactions |
| `write_err`  | Write operations with invalid address |
| `read_err`   | Read operations with invalid address |
| `reset_dut`  | Reset sequence |
| `writeb_readb` | Back-to-back write followed by read |

All sequences use **constrained randomization** and explicit operation selection.

---

### 🔹 Driver
- Drives SPI transactions through a virtual interface
- Handles reset, read, and write behavior
- Synchronizes stimulus with `done` signal
- Implements protocol-aware driving logic

---

### 🔹 Monitor
- Passively observes DUT interface signals
- Converts pin-level activity into transaction objects
- Sends transactions to the scoreboard via analysis port
- Detects reset, read, write, and error events

---

### 🔹 Scoreboard
- Maintains an internal **reference memory model**
- Verifies:
  - Correct write behavior
  - Read data integrity
  - Proper error signaling
- Reports data match/mismatch conditions

---

## 🔁 DUT Functional Behavior

### ✔ Write Operation
- Address < 32
- Serial transmission of `{data, address, write}` packet
- Memory updated upon successful completion

### ✔ Read Operation
- Address < 32
- Address phase followed by data phase
- Data returned via MISO

### ❌ Error Condition
- Address ≥ 32
- Chip select remains inactive
- Error flag asserted
- Transaction safely terminated

---

## ⏱ Clock and Reset
- Clock generated in the testbench
- Reset supported at startup and via UVM sequence
- Reset brings DUT and memory to a known state

---

## 📊 Debug and Visibility
- VCD waveform dumping enabled
- Informative `uvm_info` logs from:
  - Driver
  - Monitor
  - Scoreboard
- Easy waveform-based debugging

---

## 🏁 Running the Simulation (EDA Playground)

1. Open the EDA Playground link
2. Select a SystemVerilog simulator
3. Enable UVM support
4. Run the simulation
5. Observe logs, scoreboard output, and waveforms

---

## 🎯 What This Project Demonstrates

- Practical UVM testbench implementation
- Constrained-random stimulus generation
- SPI protocol understanding
- Scoreboard-based data verification
- Error injection and handling
- Clean separation of DUT and verification logic

---

## 🚀 Future Enhancements

- Functional coverage integration
- UVM register model support
- SPI protocol assertions (SVA)
- Multi-slave SPI support
- Parameterized data/address widths

---

## 🧠 Intended Audience

- VLSI verification students
- Entry-level verification engineers
- Learners exploring **UVM with protocol-based designs**

---

## 📎 Notes

This project is intended for **learning and demonstration purposes**, while closely following **industry-standard verification practices**.

