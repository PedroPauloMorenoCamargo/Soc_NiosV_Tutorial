# Nios V FPGA Tutorial

In this tutorial, we will create and customize a soft processor with **Nios** (an embedded system with a processor and peripherals), embed it in the FPGA, and write code to control LEDs. This is a follow-up of the Tutorial-FPGA-NIOS.

## Getting Started

To follow this tutorial, you will need:

- **Hardware**: DE10-Standard and DE0-CV
- **Software**: Quartus 23.1std(23.1.1) or later
- **Documents**: DE10-Standard_User_manual.pdf, DE0_CV_User_Manual_v1,12.pdf and AN 985:_Nios_V_Processor Tutorial.pdf

---

## Setting Up the Tools

Firstly, to start the project it's necessary to download the tools which we are going to use. Those tools are Quartus 23.1std or later and 

---

## Platform Designer (PD)

Platform Designer (formerly QSYS) is a tool provided by Intel and integrated into Quartus, enabling you to develop complex systems visually. You can add and connect IP cores to create applications quickly.

---

## Nios V

**Nios V** is a soft processor provided by Intel and integrated into Quartus. Based on a RISC-V architecture, Nios V supports the addition of custom instructions implemented in HDL. This makes it highly customizable.

---

## Creating a Simple SoC

We will now create a basic SoC with the following components:

- A clock interface
- A memory (data and program storage)
- The **Nios V** processor
- A **PIO** peripheral (for managing digital outputs)
- A **JTAG-UART** for debugging via print statements

### Step-by-Step Guide

1. **Create a New Project**:
    - Copy the `Lab1_FPGA_RTL/` folder and rename it to `Lab2_FPGA_NIOS/`.
    - Open the project in Quartus from this new folder.

2. **Open Platform Designer**:
    - In Quartus, go to **Tools** > **Platform Designer**.

3. **Add Components**:
    - Add the following components with the specified configurations:
        - **On-Chip Memory (RAM or ROM Intel FPGA IP)**
            - Type: RAM
            - Memory size: 32768 bytes
        - **JTAG UART Intel FPGA IP**
            - Default settings
        - **PIO (Parallel I/O) Intel FPGA IP**
            - Width: 6
            - Direction: Output
        - **Nios V Processor**
            - Type: Nios V/e

---

## Connecting Clock and Reset

All peripherals need to be connected to the clock and reset signals. In Platform Designer, connect all clock and reset signals to the `clk` and `clk_rst` signals of the `clk_0` peripheral.

---

## Connecting the Bus

The main communication bus in Platform Designer is the Avalon bus, which is used to connect peripherals to the Nios V processor.

---

## Writing the LED Control Code

With the SoC created, let’s write a simple C program to control the LEDs:

```c
#include <stdio.h>
#include "system.h"
#include <alt_types.h>
#include <io.h> // For Avalon read/write functions

void delay(int n) {
    volatile unsigned int delay = 0;
    while (delay < n) {
        delay++;
    }
}

int main(void) {
    unsigned int led = 0;

    printf("Nios V Tutorial: LED Control\n");

    while (1) {
        if (led < PIO_0_DATA_WIDTH) { // Use the data width defined for the PIO
            IOWR_32DIRECT(PIO_0_BASE, 0, 0x01 << led++);
            usleep(50000); // 50 ms delay
        } else {
            led = 0; // Reset LED position after reaching the limit
        }
    }

    return 0;
}
