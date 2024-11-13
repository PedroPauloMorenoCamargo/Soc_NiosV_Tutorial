# Nios V FPGA Tutorial

In this tutorial, we will create and customize a soft processor with **Nios** (an embedded system with a processor and peripherals), embed it in the FPGA, and write code to control LEDs. This is a follow-up of the [Tutorial-FPGA-NIOS](https://insper.github.io/Embarcados-Avancados/Tutorial-FPGA-NIOS/).

## Getting Started

To follow this tutorial, you will need:

- **Hardware**: DE10-Standard and DE0-CV
- **Software**: Quartus 23.1std (23.1.1) or later and Ashling RiscFree IDE
- **OS**: Windows
- **Documents**:
  - [DE10-Standard_User_manual.pdf](./pdfs/DE10-Standard_User_manual.pdf)
  - [DE0_CV_User_Manual_v1.12.pdf](./pdfs/DE0_CV_User_Manual_v1.12.pdf)
  - [AN-784468-784469.pdf](./pdfs/an-784468-784469.pdf)

---

## Setting Up the Tools

To get started with the project, you’ll need to download and install a few essential tools. These include **Quartus 23.1std (or later)** and the **Ashling RiscFree IDE**, which will be used to compile and run the project, respectively.

### Step 1: Download

If Quartus is not yet installed on your system, begin by downloading the necessary files from the [Intel Website](https://www.intel.com/content/www/us/en/software-kit/825278/intel-quartus-prime-lite-edition-design-software-version-23-1-1-for-windows.html). Ensure you download the following files:

- **Quartus Download**: ![Quartus Download](./imgs/Quartus_download.png)
- **Cyclone V Download**: ![Cyclone V Download](./imgs/Cyclonev_Download.png)
- **Ashling RiscFree IDE Download**: ![Ashling RiscFree IDE Download](./imgs/Aishling_Download.png)
- **Quartus Build**: ![Quartus Build](./imgs/Quartus_Build.png)

Once downloaded, place all files in the same folder, as shown below:

**Image of files grouped together**: ![Grouped Files](./imgs/Quartus_Build.png)

Now, run the `QuartusSetup.exe` file to initiate the installation process.

If Quartus is already installed, you can skip to this step. Download the **Ashling RiscFree IDE** and run the RiscFree executable to set it up as an external tool.



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
