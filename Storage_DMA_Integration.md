
# 📂 Data Conversion & Storage - 8086 Microprocessor Project  
### Giovanny Garcia

This document outlines the implementation of the ADC/DAC interface, floppy disk storage using the 8272 controller, and DMA-based high-speed data integration using the 8237 DMA controller in the 8086 microprocessor system.

---

## 🧠 Overview

This section describes the implementation of data conversion (ADC/DAC), floppy disk storage control using the 8272, and high-speed data transfer via the 8237 DMA controller. Integration focuses on efficient data movement across peripherals using DMA and aligning all operations within the 8086 memory and I/O architecture.

---

## 🔌 Hardware Components

### 1. Analog-to-Digital Converter (ADC0808)
- Converts analog signals into digital.
- Mapped at: `0x0200` (Data), `0x0202` (Status)

### 2. Digital-to-Analog Converter (DAC0800)
- Converts digital data into analog signals.
- Mapped at: `0x0208` (Data), `0x020A` (Control)

### 3. 8272 Floppy Disk Controller (FDC)
- Manages read/write operations to floppy disks.
- Registers: Status `0x0320`, Command `0x0322`, Data `0x0323`

### 4. 8237 DMA Controller
- Transfers data between memory and peripherals with minimal CPU involvement.
- I/O range: `0x0000–0x000F` (standard)

---

## 🖼️ Block Diagrams

### Data Conversion System

```
[Analog Sensor] --> [ADC0808 @0x0200] --> [8237 DMA] --> [Memory 0x40000]
                                                      |
                                                      v
                                                [DAC0800 @0x0208] <-- [Memory]
```

### Floppy Storage with DMA

```
[ADC0808] --> [DMA Ch1] --> [Memory @0x40000] --> [DMA Ch2] --> [8272 FDC @0x0322] --> [Floppy Disk]
```

---

## ⚙️ Register-Level Interface

### ADC & DAC

| Device    | Address Range | Register         |
|-----------|----------------|------------------|
| ADC0808   | 0x0200–0x0207  | Data: 0x0200     |
|           |                | Status: 0x0202   |
| DAC0800   | 0x0208–0x020F  | Data: 0x0208     |
|           |                | Control: 0x020A  |

### 8272 Floppy Controller

| Register | Address |
|----------|---------|
| Status   | 0x0320  |
| Command  | 0x0322  |
| Data     | 0x0323  |

---

## 📜 Pseudocode Examples

### 1. ADC to Memory using DMA (Ch1)

```pseudocode
// Setup DMA channel 1 for ADC read
configure_dma(channel=1, source=0x0200, dest=0x40000, count=512, mode=READ)

// Start ADC conversion
write_port(0x020A, START)

// Enable DMA channel
enable_dma_channel(1)

// Wait for completion
wait_dma_done(channel=1)
```

---

### 2. Memory to DAC using DMA (Ch2)

```pseudocode
// Setup DMA for DAC write
configure_dma(channel=2, source=0x40000, dest=0x0208, count=512, mode=WRITE)

// Enable DAC output
write_port(0x020A, ENABLE)

// Enable DMA channel
enable_dma_channel(2)

// Wait for completion
wait_dma_done(channel=2)
```

---

### 3. ADC to Floppy via DMA

```pseudocode
// ADC --> DMA --> RAM
configure_dma(channel=1, source=0x0200, dest=0x40000, count=1024, mode=READ)
start_adc()
enable_dma_channel(1)
wait_dma_done(1)

// RAM --> 8272 via DMA
configure_dma(channel=2, source=0x40000, dest=0x0323, count=1024, mode=WRITE)
write_port(0x0322, CMD_WRITE_SECTOR)
enable_dma_channel(2)
wait_dma_done(2)
```

---

## 🧩 Integration Notes

- **DMA Buffer Assignment**:
  - ADC Data: `0x40000–0x40FFF` 
  - USB Buffer: `0x41000–0x41FFF`

- **Interrupt Assignments**:
  - FDC: Uses `IRQ6` → Vector `0x0E`
  - USB: Uses `IRQ5` → Vector `0x0D`

- **DMA Channels**:
  - Ch1: Shared ADC
  - Ch2: Floppy Controller
  - Ch3: USB 

- **I/O Addressing**:
  - Follows updated team map (no conflicts)
  - Fully coordinated with address updates

---

## ✅ Summary

This subsystem delivers efficient data acquisition and storage using DMA to minimize CPU load. All components use team-standard addresses and buffer spaces, and integrate seamlessly with USB and interrupt systems. DMA channels and IRQs have been cleanly divided for maximum performance and zero conflict.
