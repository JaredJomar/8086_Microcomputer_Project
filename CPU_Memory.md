# CPU & MEMORY ARCHITECTURE - JARED: COMPLETE SYSTEM DESIGN

This document presents the complete design of an 8086-based microcomputer system with 1MB memory, coprocessor 8087, and comprehensive I/O interfaces including diskette controller 8272 for teams of 4 members.

## 1. GENERAL BLOCK DIAGRAM OF THE SYSTEM

The system architecture follows a hierarchical design with the 8086 CPU as the central processing unit, supported by the 8087 coprocessor for floating-point operations. All components communicate through a unified system bus structure.

```
                    8086 MICROCOMPUTER SYSTEM - 10MHz
    ┌─────────────────────────────────────────────────────────────────────────────┐
    │                           SYSTEM BUS                                        │
    └─────────────────────────────────────────────────────────────────────────────┘
           │                    │                    │                    │
    ┌──────▼──────┐    ┌──────▼──────┐    ┌──────▼──────┐    ┌──────▼──────┐
    │             │    │             │    │             │    │             │
    │ PROCESSOR   │    │   MEMORY    │    │ CONTROLLER  │    │ PERIPHERALS │
    │             │    │             │    │ INTERRUPTS  │    │             │
    └─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘

### PROCESSOR
┌─────────────────────────────────────────┐
│            CPU 8086 (10MHz)             │
│  ┌─────────────┐  ┌─────────────────┐   │
│  │    ALU      │  │ REGISTERS       │   │
│  │             │  │ AX,BX,CX,DX     │   │
│  │             │  │ SI,DI,BP,SP     │   │
│  └─────────────┘  │ CS,DS,ES,SS     │   │
│                   └─────────────────┘   │
└─────────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────┐
│      COPROCESSOR 8087 (10MHz)           │
│  ┌─────────────┐  ┌─────────────────┐   │
│  │   FPU       │  │ FLOAT REGISTERS │   │
│  │  (80 bits)  │  │ ST(0) - ST(7)   │   │
│  │             │  │                 │   │
│  └─────────────┘  └─────────────────┘   │
└─────────────────────────────────────────┘

### BUS SYSTEM
                    20 bits
          ┌──────────────────────────┐
          │   ADDRESS BUS            │
          │    (A0 - A19)            │
          └──────────────────────────┘
                     │
          ┌──────────▼───────────────┐
          │   DATA BUS               │
          │    (D0 - D15)            │
          │     16 bits              │
          └──────────────────────────┘
                     │
          ┌──────────▼──────────────┐
          │   CONTROL BUS           │
          │  RD, WR, M/IO, DT/R     │
          │  ALE, DEN, READY        │
          └─────────────────────────┘

### MAIN MEMORY (1 MBYTE)
┌─────────────────────────────────────────┐
│              RAM (768KB)                │
│         Addresses: 00000h-BFFFFh        │
├─────────────────────────────────────────┤
│              ROM (128KB)                │
│         Addresses: E0000h-FFFFFh        │
├─────────────────────────────────────────┤
│            I/O AREA (128KB)             │
│         Addresses: C0000h-DFFFFh        │
└─────────────────────────────────────────┘
```

**System Bus Explanation:**
The 8086 provides a 20-bit address bus allowing access to 1MB of memory space. The 16-bit data bus enables efficient data transfer, while control signals coordinate memory and I/O operations. The coprocessor 8087 operates in parallel with the CPU for mathematical computations.

## 2. DETAILED I/O INTERFACE DIAGRAMS

### 2.1 INTERRUPT CONTROLLER (8259A) - Address: C0000h-C00FFh
```
┌─────────────────────────────────────────────────────────────┐
│                    8259A INTERRUPT CONTROLLER               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │   IRQ0-7     │  │  PRIORITY    │  │   CONTROL    │       │
│  │   INPUTS     │  │   LOGIC      │  │   REGISTER   │       │
│  │              │  │              │  │              │       │
│  └──────────────┘  └──────────────┘  └──────────────┘       │
│         │                    │                    │         │
│         ▼                    ▼                    ▼         │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              TO CPU INTR PIN                        │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘

Connected IRQ Lines:
- IRQ0: Timer (system clock)
- IRQ1: Keyboard
- IRQ2: Serial port
- IRQ3: Parallel port  
- IRQ4: USB controller
- IRQ5: ADC/DAC
- IRQ6: Floppy controller
- IRQ7: Printer
```

### 2.2 DMA CONTROLLER (8237) - Address: C0100h-C01FFh
```
┌─────────────────────────────────────────────────────────────┐
│                    8237 DMA CONTROLLER                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │   CHANNEL 0  │  │   CHANNEL 1  │  │   CHANNEL 2  │       │
│  │   FLOPPY     │  │   USB        │  │   RESERVED   │       │
│  └──────────────┘  └──────────────┘  └──────────────┘       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │   CHANNEL 3  │  │   ADDRESS    │  │   CONTROL    │       │
│  │   RESERVED   │  │   REGISTERS  │  │   LOGIC      │       │
│  └──────────────┘  └──────────────┘  └──────────────┘       │
└─────────────────────────────────────────────────────────────┘

DMA Channels:
- CH0: Floppy disk high-speed transfers
- CH1: USB data transfers  
- CH2: Available for expansion
- CH3: Available for expansion
```

### 2.3 SERIAL PORT (8251 UART) - Address: C0200h-C02FFh
```
┌─────────────────────────────────────────────────────────────┐
│                    8251 UART (RS-232)                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │   TX BUFFER  │  │   RX BUFFER  │  │   BAUD RATE  │       │
│  │              │  │              │  │   GENERATOR  │       │
│  └──────────────┘  └──────────────┘  └──────────────┘       │
│         │                    │                    │         │
│         ▼                    ▼                    ▼         │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              RS-232 CONNECTOR                       │    │
│  │         TXD, RXD, RTS, CTS, DTR, DSR                │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘

Features:
- Programmable baud rates (110-9600 bps)
- Full duplex communication
- Hardware handshaking support
```

### 2.4 PARALLEL PORT (8255 PPI) - Address: C0300h-C03FFh
```
┌─────────────────────────────────────────────────────────────┐
│                    8255 PARALLEL PORT                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │   PORT A     │  │   PORT B     │  │   PORT C     │       │
│  │   (8 bits)   │  │   (8 bits)   │  │   (8 bits)   │       │
│  └──────────────┘  └──────────────┘  └──────────────┘       │
│         │                    │                    │         │
│         ▼                    ▼                    ▼         │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              CENTRONICS CONNECTOR                   │    │
│  │         DATA[0-7], STROBE, ACK, BUSY                │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘

Configuration:
- Port A: Data output (to printer)
- Port B: Status input (from printer)  
- Port C: Control signals
```

### 2.5 KEYBOARD INTERFACE (8279) - Address: C0400h-C04FFh
```
┌─────────────────────────────────────────────────────────────┐
│                    8279 KEYBOARD CONTROLLER                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │   SCAN       │  │   KEY CODE   │  │   FIFO       │       │
│  │   COUNTER    │  │   DECODER    │  │   BUFFER     │       │
│  └──────────────┘  └──────────────┘  └──────────────┘       │
│         │                    │                    │         │
│         ▼                    ▼                    ▼         │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              64-KEY MATRIX                          │    │
│  │         8 Rows x 8 Columns = 64 Keys                │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘

Features:
- 64-key matrix scanning
- Key debouncing
- FIFO buffer for key codes
- Interrupt generation on key press
```

### 2.6 DISPLAY INTERFACE (8279) - Address: C0500h-C05FFh  
```
┌─────────────────────────────────────────────────────────────┐
│                    8279 DISPLAY CONTROLLER                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │   DISPLAY    │  │   SEGMENT    │  │   DIGIT      │       │
│  │   RAM        │  │   DECODER    │  │   DRIVERS    │       │
│  └──────────────┘  └──────────────┘  └──────────────┘       │
│         │                    │                    │         │
│         ▼                    ▼                    ▼         │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              16-DIGIT 7-SEGMENT DISPLAY             │    │
│  │         Common Cathode Configuration                │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘

Features:
- 16 digits x 7 segments
- Automatic multiplexing
- Brightness control
- Hexadecimal display capability
```

### 2.7 ADC/DAC INTERFACE - Address: C0600h-C06FFh
```
┌─────────────────────────────────────────────────────────────┐
│                    ADC/DAC INTERFACE                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │   ADC 0804   │  │   DAC 0808   │  │   CONTROL    │       │
│  │   8-bit      │  │   8-bit      │  │   LOGIC      │       │
│  └──────────────┘  └──────────────┘  └──────────────┘       │
│         │                    │                    │         │
│         ▼                    ▼                    ▼         │
│  ┌─────────────────────────────────────────────────────┐    │
│  │          ANALOG I/O CONNECTORS                      │    │
│  │    Analog Input: 0-5V    Analog Output: 0-5V        │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘

Specifications:
- ADC: 8-bit resolution, 0-5V input range
- DAC: 8-bit resolution, 0-5V output range
- Conversion time: <100μs
```

### 2.8 USB CONTROLLER + DMA - Address: C0700h-C07FFh
```
┌─────────────────────────────────────────────────────────────┐
│                    USB CONTROLLER                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │   USB HOST   │  │   DMA        │  │   BUFFER     │       │
│  │   CONTROLLER │  │   INTERFACE  │  │   MEMORY     │       │
│  └──────────────┘  └──────────────┘  └──────────────┘       │
│         │                    │                    │         │
│         ▼                    ▼                    ▼         │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              USB CONNECTOR (Type A)                 │    │
│  │         High-speed data transfer via DMA            │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘

Features:
- USB 1.1 compatible
- DMA channel 1 for high-speed transfers
- Interrupt-driven operation
```

### 2.9 PRINTER CONTROLLER - Address: C0800h-C08FFh
```
┌─────────────────────────────────────────────────────────────┐
│                    PRINTER CONTROLLER                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │   DATA       │  │   STATUS     │  │   CONTROL    │       │
│  │   BUFFER     │  │   MONITOR    │  │   SIGNALS    │       │
│  └──────────────┘  └──────────────┘  └──────────────┘       │
│         │                    │                    │         │
│         ▼                    ▼                    ▼         │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              PRINTER CONNECTOR                      │    │
│  │         Parallel interface (Centronics)             │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘

Features:
- Centronics parallel interface
- Automatic paper feed control
- Ready/Busy status monitoring
```

### 2.10 FLOPPY CONTROLLER 8272 - Address: C0900h-C09FFh
```
┌─────────────────────────────────────────────────────────────┐
│                    8272 FLOPPY CONTROLLER                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │   COMMAND    │  │   DATA       │  │   STATUS     │       │
│  │   PROCESSOR  │  │   SEPARATOR  │  │   REGISTER   │       │
│  └──────────────┘  └──────────────┘  └──────────────┘       │
│         │                    │                    │         │
│         ▼                    ▼                    ▼         │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              FLOPPY DRIVE INTERFACE                 │    │
│  │    Step, Direction, Write Gate, Read Data           │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘

Features:
- Support for 5.25" and 3.5" drives
- Double density (360KB/720KB)
- DMA channel 0 for data transfers
- Built-in formatting capability
```

### CONNECTED PERIPHERALS
```
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│   64-KEYBOARD   │  │  7-SEG DISPLAY  │  │   PRINTER       │
│    64 KEYS      │  │   16 DIGITS     │  │   PARALLEL      │
└─────────────────┘  └─────────────────┘  └─────────────────┘

┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│  SERIAL PORT    │  │ PARALLEL PORT   │  │  USB + DMA      │
│    RS-232       │  │   CENTRONICS    │  │                 │
└─────────────────┘  └─────────────────┘  └─────────────────┘

┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│   ADC (8 bit)   │  │   DAC (8 bit)   │  │ FLOPPY 8272     │
│                 │  │                 │  │                 │
└─────────────────┘  └─────────────────┘  └─────────────────┘
```

## 3. DETAILED MEMORY MAP

The memory organization follows a structured approach with dedicated areas for different system functions. The 1MB address space is efficiently divided between RAM for user programs and data, ROM for system firmware, and I/O space for peripheral interfaces.

```
MEMORY MAP - 8086 SYSTEM (1 MBYTE)
Physical Addresses (20 bits) = 00000h to FFFFFh

┌──────────────┬──────────────┬─────────────┬────────────────────────────┐
│  ADDRESS     │   SIZE       │   TYPE      │      DESCRIPTION           │
├──────────────┼──────────────┼─────────────┼────────────────────────────┤
│ 00000h-003FFh│   1KB        │   RAM       │ Interrupt Vector Table     │
│ 00400h-3FFFFh│ 255KB        │   RAM       │ User Program Area          │
│ 40000h-7FFFFh│ 256KB        │   RAM       │ User Data Area             │
│ 80000h-BFFFFh│ 256KB        │   RAM       │ System Buffer/Stack        │
│ C0000h-CFFFFh│  64KB        │   I/O       │ Peripheral Mapping Area    │
│ D0000h-DFFFFh│  64KB        │ RESERVED    │ Reserved for Video & I/O   │
│ E0000h-F7FFFh│  96KB        │   ROM       │ System BIOS                │
│ F8000h-FFFFFh│  32KB        │   ROM       │ Boot BIOS                  │
└──────────────┴──────────────┴─────────────┴────────────────────────────┘

I/O AREA DETAILS (C0000h - CFFFFh):
┌──────────────┬─────────────┬────────────────────────────────────┐
│  ADDRESS     │   SIZE      │         DEVICE                     │
├──────────────┼─────────────┼────────────────────────────────────┤
│ C0000h-C00FFh│   256B      │ Interrupt Controller (8259A)       │
│ C0100h-C01FFh│   256B      │ DMA Controller (8237)              │
│ C0200h-C02FFh│   256B      │ Serial Port (8251 UART)            │
│ C0300h-C03FFh│   256B      │ Parallel Port (8255 PPI)           │
│ C0400h-C04FFh│   256B      │ Matrix Keyboard (8279)             │
│ C0500h-C05FFh│   256B      │ 7-Segment Display (8279)           │
│ C0600h-C06FFh│   256B      │ ADC/DAC Interface                  │
│ C0700h-C07FFh│   256B      │ USB Controller                     │
│ C0800h-C08FFh│   256B      │ Printer Controller                 │
│ C0900h-C09FFh│   256B      │ Floppy Controller (8272)           │
│ C0A00h-CFFFFh│  ~62KB      │ Reserved for Future Expansion      │
└──────────────┴─────────────┴────────────────────────────────────┘

RESERVED AREA DETAILS (D0000h - DFFFFh):
┌──────────────┬─────────────┬────────────────────────────────────┐
│  ADDRESS     │   SIZE      │         PURPOSE                    │
├──────────────┼─────────────┼────────────────────────────────────┤
│ D0000h-D7FFFh│   32KB      │ Reserved for Video Memory          │
│ D8000h-DFFFFh│   32KB      │ Reserved for Additional I/O        │
└──────────────┴─────────────┴────────────────────────────────────┘
```

## 4. ADDRESS DECODING SYSTEM

The address decoding system uses a hierarchical approach with a main decoder (74138) that examines the four most significant address bits (A19-A16) to select between memory and I/O regions. This design provides clear separation between different system areas and allows for easy expansion.

```
ADDRESS DECODER - HIERARCHICAL DESIGN

                    A19-A16 (4 MSB bits)
                         │
            ┌────────────▼────────────┐
            │    MAIN DECODER         │
            │      (74138)            │
            └─────────────────────────┘
                 │    │    │    │
        ┌────────┘    │    │    └────────┐
        │             │    │             │
    ┌───▼───┐    ┌───▼───┐ │      ┌─────▼─────┐
    │  RAM  │    │  I/O  │ │      │    ROM    │
    │ BANK  │    │ AREA  │ │      │   BANK    │
    └───────┘    └───────┘ │      └───────────┘
                           │
                    ┌─────▼─────┐
                    │ RESERVED  │
                    └───────────┘

RAM DECODING (00000h-BFFFFh):
- Uses A19-A18 = 00, 01, 10
- Chips: 4 x DRAM 256KB (4164 or similar)
- Configuration: 2 banks x 2 chips each
- Row/column address multiplexing

ROM DECODING (E0000h-FFFFFh):
- Uses A19-A17 = 111
- Chips: 4 x EPROM 32KB (27256)
- Direct addressing A16-A0

I/O DECODING (C0000h-DFFFFh):
- Uses A19-A17 = 110
- Secondary decoder (74138)
- A16-A12 selects specific device
```

### ADDRESS DECODER – DETAILED PIN CONNECTIONS

#### PRIMARY DECODER (74138)

| Address Line | 74138 Input Pin |
|--------------|-----------------|
| A19          | A               |
| A18          | B               |
| A17          | C               |

**74138 Output Selection:**

- **Y0 (00x):** RAM Bank 0 (00000h–3FFFFh)
- **Y1 (01x):** RAM Bank 1 (40000h–7FFFFh)
- **Y2 (10x):** RAM Bank 2 (80000h–BFFFFh)
- **Y3 (110):** I/O Area (C0000h–DFFFFh)
- **Y4 (111):** ROM Area (E0000h–FFFFFh)

#### MEMORY BANK SELECTION

**RAM Banks (A19–A18):**

- `00`: Bank 0 (256KB) – User Program
- `01`: Bank 1 (256KB) – User Data
- `10`: Bank 2 (256KB) – System/Stack
- `11`: Not used for RAM

**I/O & ROM Selection (A19–A17):**

- `110`: I/O Area
- `111`: ROM Area

#### SECONDARY DECODER (74138 for I/O)

Input: **A16–A12** for device selection

| A16–A12 | Y Output | Device    |
|---------|----------|-----------|
| 00000   | Y0       | 8259A     |
| 00001   | Y1       | 8237 DMA  |
| 00010   | Y2       | 8251 UART |
| 00011   | Y3       | 8255 PPI  |
| 00100   | Y4       | Keyboard  |
| 00101   | Y5       | Display   |
| 00110   | Y6       | ADC/DAC   |
| 00111   | Y7       | USB       |

## 5. PHYSICAL MEMORY CONFIGURATION

The physical memory implementation uses a combination of dynamic RAM for main memory and EPROM for system firmware. The DRAM configuration includes refresh circuitry to maintain data integrity, while the ROM provides non-volatile storage for BIOS and boot routines.

```
MEMORY CHIP CONFIGURATION

RAM - DYNAMIC DRAM CONFIGURATION:
┌─────────────────────────────────────────────────────────────┐
│                    BANK 0 (256KB)                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │   DRAM 0     │  │   DRAM 1     │  │   DRAM 2     │       │
│  │   64K x 1    │  │   64K x 1    │  │   64K x 1    │       │
│  │   4164       │  │   4164       │  │   4164       │       │
│  └──────────────┘  └──────────────┘  └──────────────┘       │
│  ┌──────────────┐  ┌──────────────┐                         │
│  │   DRAM 3     │  │  RAS/CAS     │                         │
│  │   64K x 1    │  │  CONTROL     │                         │
│  │   4164       │  │  74170       │                         │
│  └──────────────┘  └──────────────┘                         │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                    BANK 1 (256KB)                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │   DRAM 4     │  │   DRAM 5     │  │   DRAM 6     │       │
│  │   64K x 1    │  │   64K x 1    │  │   64K x 1    │       │
│  │   4164       │  │   4164       │  │   4164       │       │
│  └──────────────┘  └──────────────┘  └──────────────┘       │
│  ┌──────────────┘  ┌──────────────┐                         │
│  │   DRAM 7     │  │  REFRESH     │                         │
│  │   64K x 1    │  │  TIMER       │                         │
│  │   4164       │  │  74123       │                         │
│  └──────────────┘  └──────────────┘                         │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                    BANK 2 (256KB)                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │   DRAM 8     │  │   DRAM 9     │  │   DRAM 10    │       │
│  │   64K x 1    │  │   64K x 1    │  │   64K x 1    │       │
│  │   4164       │  │   4164       │  │   4164       │       │
│  └──────────────┘  └──────────────┘  └──────────────┘       │
│  ┌──────────────┐  ┌──────────────┐                         │
│  │   DRAM 11    │  │  ADDRESS     │                         │
│  │   64K x 1    │  │  DECODER     │                         │
│  │   4164       │  │  74138       │                         │
│  └──────────────┘  └──────────────┘                         │
└─────────────────────────────────────────────────────────────┘

Note: Each bank uses four 4164 DRAM chips (64K x 1 bit) to provide 256KB with 
16-bit data bus width. All banks use identical chip configurations for
uniformity and simplified control logic.

ROM - EPROM CONFIGURATION:
┌─────────────────────────────────────────────────────────────┐
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │   EPROM 0    │  │   EPROM 1    │  │   EPROM 2    │       │
│  │   32KB       │  │   32KB       │  │   32KB       │       │
│  │   27256      │  │   27256      │  │   27256      │       │
│  │  MAIN BIOS   │  │  EXT BIOS    │  │  BOOT LOAD   │       │
│  └──────────────┘  └──────────────┘  └──────────────┘       │
└─────────────────────────────────────────────────────────────┘

MEMORY CONTROL SIGNALS:
┌─────────────┬────────────────────────────────────────┐
│ Signal      │ Function                               │
├─────────────┼────────────────────────────────────────┤
│ RAS         │ Row Address Strobe (DRAM timing)       │
│ CAS         │ Column Address Strobe (DRAM timing)    │
│ WE          │ Write Enable (active low)              │
│ OE          │ Output Enable (ROM/RAM read)           │
│ CS          │ Chip Select (from address decoder)     │
└─────────────┴────────────────────────────────────────┘
```

## 6. SYSTEM IMPLEMENTATION AND CODE REFERENCES

### 6.1 Complete System Integration

This microcomputer system, as documented and implemented by the team, fulfills all requirements for Part I-B (teams of 4 members):

* ✅ **8086 CPU** at 10 MHz with **8087 coprocessor**
* ✅ **1MB memory** (768 KB RAM + 128 KB ROM + 128 KB mapped I/O area)
* ✅ **Parallel connector** (8255 PPI)
* ✅ **Serial connector** (8251 UART)
* ✅ **ADC and DAC** interfaces for data conversion
* ✅ **16-digit seven-segment display** (8279)
* ✅ **64-key matrix keyboard** (8279)
* ✅ **USB controller with DMA** for high-speed transfers
* ✅ **Printer connector** (Centronics/parallel interface)
* ✅ **Interrupt controller** (8259A) managing I/O priorities
* ✅ **Diskette controller** (8272) for floppy disk operations
* ✅ **DMA controller** (8237) with 4 channels for peripheral and memory transfers

All system components are represented in the architectural diagrams, memory map, and explanations throughout [`CPU_Memory.md`](CPU_Memory.md).

### 6.2 Programming and Code Files

The complete system initialization, memory management, and driver logic is provided across the following files:

**Assembly Code:** [`CPU_Memory.asm`](CPU_Memory.asm)
* System initialization routines and structure (with documented procedure stubs)
* Segment and stack configuration
* Memory and coprocessor test pattern setup
* Team base address definitions for all peripherals

**Pseudocode Documentation:** [`CPU_Memory_pseudocode.md`](CPU_Memory_pseudocode.md)
* Step-by-step pseudocode for system and memory initialization
* Coprocessor (8087) initialization and test routines
* Memory bank configuration and ROM verification
* Address mapping and coordination between team members
* Standard memory read/write/copy routines

For a full overview of architecture and interface details, refer to the block diagrams, memory map, and explanations throughout [`CPU_Memory.md`](CPU_Memory.md).

### 6.3 Team Address Coordination

To enable clear collaboration and avoid address conflicts, the I/O address space is explicitly partitioned for each team member as follows:

| **Team Member** | **Devices**                             | **Address Range(s)**         | **Allocated Size** |
| --------------- | --------------------------------------- | ---------------------------- | ------------------ |
| Person 2        | Keyboard, Display, Printer              | C0400h–C05FFh, C0800h–C08FFh | 512 B              |
| Person 3        | Interrupts, Serial, Parallel, USB + DMA | C0000h–C03FFh, C0700h–C07FFh | 1280 B             |
| Person 4        | ADC/DAC, Floppy Controller (8272)       | C0600h–C06FFh, C0900h–C09FFh | 512 B              |

**Note:** All assignments match the memory map and decoding diagrams; see the I/O details in Section 2 and the pseudocode address table for further reference.

### 6.4 System Specifications Summary

* **CPU:** Intel 8086 @ 10 MHz
* **Coprocessor:** Intel 8087 (floating-point)
* **Total Memory:** 1 MB (768 KB RAM, 128 KB ROM, 128 KB I/O mapped)
* **Main Peripherals:** 8255 PPI (parallel), 8251 UART (serial), 8259A (interrupt), 8237 (DMA), 8279 (display/keyboard), ADC 0804, DAC 0808, USB interface, printer, 8272 floppy controller
* **DMA Channels:** 4 (8237)
* **Interrupt Levels:** 8 (8259A)
* **Storage:** 8272 floppy disk controller (5.25"/3.5" drives, double density)
* **User I/O:** 64-key keyboard, 16-digit 7-segment display
* **Analog I/O:** 8-bit ADC and DAC
* **Modern Expansion:** USB with DMA support
## 7. 8086/8087 PHYSICAL INTEGRATION

### 7.1 Signal Connections and Bus Sharing

```
┌────────────────────────────────────────────────────────┐
│           8086/8087 PHYSICAL CONNECTIONS               │
│                                                        │
│  8086 Signal     8087 Signal     Function              │
│  ────────────    ────────────    ──────────────────    │
│     TEST/          BUSY/         Coprocessor Status    │
│     QS0,QS1       QS0,QS1       Queue Status           │
│     S0-S2         S0-S2         Bus Cycle Status       │
│     CLK           CLK           10MHz System Clock     │
│     READY         READY         Wait State Control     │
│     RQ/GT0        REQUEST       Bus Request/Grant      │
└────────────────────────────────────────────────────────┘
```

### 7.2 Bus Arbitration and Timing

1. **Bus Control**
   - 8087 monitors all bus cycles through status lines
   - Uses RQ/GT protocol for bus access
   - Maintains instruction synchronization via queue status

2. **Clock Synchronization**
   - Both processors share 10MHz clock
   - CLK signal distributed with minimal skew
   - Maximum 100ns cycle time at full speed

3. **Wait State Generation**
   - System can insert wait states via READY signal
   - Typical memory cycle: 4 clock periods
   - Wait states added for slower peripherals

### 7.3 Memory Access Timing (10MHz Operation)

**Basic Read/Write Cycle Timing Diagram:**

```
CLK    ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐
       │ │ │ │ │ │ │ │ │ │ │ │
       └─┘ └─┘ └─┘ └─┘ └─┘ └─┘
        T1  T2  T3  T4  Tw  T4

ALE    ┌───┐
       │   │
       └───┘

A19-A0 ──┬───────────────┐
         │   Valid      │
         └──────────────┘

CS,RD   ───┬─────────────┐
           │  Active Low │
           └─────────────┘

DATA    ────────┬───────┐
                │Valid  │
                └───────┘

READY   ──────────┐ ┌───┘
                  └─┘
```

**Timing Notes:**
- **T1–T4:** Standard bus cycle states.
- **Tw:** Optional wait state inserted if READY is not asserted.
- **ALE:** Address Latch Enable pulse at start of cycle.
- **A19–A0:** Address lines valid after ALE.
- **CS, RD:** Chip Select and Read/Write signals active low during access.
- **DATA:** Data lines valid during T3/Tw.
- **READY:** If low, CPU inserts wait state(s) (Tw) before completing cycle.

Note: The timing diagram shows a typical memory access with one wait 
state (Tw). The system can operate at 10MHz with proper wait state 
insertion for different memory and I/O devices.
