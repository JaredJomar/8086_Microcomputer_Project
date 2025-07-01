## 👥 Task Division for 4 Members (with File Locations Updated)

### Overview

### Jared: CPU, Coprocessor, Memory & Map

- Block diagram for **8086**, **8087**, and overall architecture.
- **Memory map:** 1MB organization, memory types, decoding.
- **Address decoding logic** for memory.
- **Relevant PDFs:** 1, 4, 6, 7
- **Location:**
    Add your work to **`CPU_Memory.md`**, and include code in:
    - `CPU_Memory.asm` (assembly code)
    - `CPU_Memory_pseudocode.md` (pseudocode)
    - `CPU_Memory.txt` (optional: summaries/raw code)
    


---


<div align="center">

<details open>
<summary><strong>🎯 <span style="font-size:1.3em">WORK COMPLETED</span></strong></summary>

<br>

<table>
  <tr>
    <th align="center">🏗️ System Architecture Design</th>
  </tr>
  <tr>
    <td align="left">
      <ul>
        <li><b>Complete 8086 system block diagram</b> with all major components</li>
        <li><b>8087 coprocessor integration</b> for floating-point operations</li>
        <li><b>System bus architecture</b> (20-bit address, 16-bit data, control signals)</li>
        <li><b>Hierarchical component organization</b> showing processor, memory, controllers, and peripherals</li>
      </ul>
    </td>
  </tr>
  <tr>
    <th align="center">💾 Memory System Implementation</th>
  </tr>
  <tr>
    <td align="left">
      <ul>
        <li><b>1MB memory organization</b> with clear address ranges:
          <ul>
            <li>768KB RAM (user programs and data)</li>
            <li>128KB ROM (system BIOS)</li>
            <li>128KB I/O mapped space (peripherals)</li>
          </ul>
        </li>
        <li><b>Complete memory map</b> with specific address ranges for each component</li>
        <li><b>Address decoding logic</b> using 74138 decoders for memory and I/O selection</li>
        <li><b>Physical memory configuration</b> with DRAM and EPROM chip specifications</li>
      </ul>
    </td>
  </tr>
  <tr>
    <th align="center">🔌 I/O System Coordination</th>
  </tr>
  <tr>
    <td align="left">
      <ul>
        <li><b>Complete I/O address space allocation</b> for all 10 peripheral devices</li>
        <li><b>Team address coordination</b> preventing conflicts between members</li>
        <li><b>Peripheral interface definitions</b> for all controllers (8259A, 8237, 8251, 8255, 8279, 8272, etc.)</li>
        <li><b>Expansion area reservation</b> for future system growth</li>
      </ul>
    </td>
  </tr>
  <tr>
    <th align="center">📝 Software Implementation</th>
  </tr>
  <tr>
    <td align="left">
      <ul>
        <li><b>Assembly language initialization routines</b> with complete system startup</li>
        <li><b>Pseudocode documentation</b> for all major system functions</li>
        <li><b>Memory management routines</b> (read, write, copy, fill operations)</li>
        <li><b>8087 coprocessor initialization</b> with detection and testing</li>
        <li><b>Team coordination functions</b> with address mapping</li>
      </ul>
    </td>
  </tr>
</table>

</details>

</div>

---

### Person 2: Parallel/Serial, Printer, Keyboard/Display

- **8255** (parallel), **8251** (serial), printer interface.
- **8279** (keyboard/display).
- Block diagrams for these peripherals and their connections.
- **Relevant PDFs:** 1, 2, 5
- **Location:** Add your work to **`IO_Peripherals.md`** (create if it doesn't exist).

---

### Person 3: Data Conversion, USB (DMA), Interrupts

- **ADC**, **DAC** interfacing (block diagrams, connections).
- **USB interface** through DMA (**8237**).
- **8259** (interrupt controller) integration for I/O.
- **Relevant PDFs:** 1, 2, 3, 5
- **Location:** Add your work to **`Data_Conversion_Interrupts.md`** (create if it doesn't exist).

---

### Person 4: Diskette Controller (8272), DMA, Integration

- Diskette controller (floppy, **8272**), **8237 DMA** for high-speed data.
- Pseudocode for **USB** and diskette transfers using DMA.
- Help integrate all I/O into final system.
- **Relevant PDFs:** 3, 5, 6
- **Location:** Add your work to **`Storage_DMA_Integration.md`** (create if it doesn't exist).

---
## 📋 Task Division Summary for 4 Members

| **Member** | **Area** | **Components** | **Deliverables** | **File(s) Needed** | **Status** |
|------------|----------|----------------|------------------|-------------------|------------|
| 🟢 **Jared** | CPU & Memory Architecture | • 8086 CPU<br>• 8087 Coprocessor<br>• 1MB RAM/ROM<br>• Address decoding | • Block diagrams<br>• Memory map<br>• Initialization code<br>• Assembly routines | **CPU_Memory.md**<br>CPU_Memory.asm<br>CPU_Memory_pseudocode.md<br>CPU_Memory.txt | **✅ COMPLETE** |
| 🟡 **Member 2** | User I/O Interface | • 16-digit 7-segment display<br>• 64-key matrix keyboard<br>• Printer | • Display driver<br>• Keyboard scanner<br>• Printer interface<br>• Assembly examples | **IO_Peripherals.md** | ⏳ PENDING |
| 🟡 **Member 3** | Communications & Interrupts | • RS-232 serial port<br>• Parallel port<br>• USB+DMA<br>• 8259A interrupt controller | • Communication drivers<br>• DMA controller<br>• Interrupt handlers<br>• USB routines | **Data_Conversion_Interrupts.md** | ⏳ PENDING |
| 🟡 **Member 4** | Data Conversion & Storage | • ADC (Analog-to-Digital)<br>• DAC (Digital-to-Analog)<br>• 8272 Floppy controller | • ADC/DAC drivers<br>• Disk controller<br>• Conversion routines<br>• Storage examples | **Storage_DMA_Integration.md** | ⏳ PENDING |
