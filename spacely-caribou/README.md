# Spacely-Caribou

Spacely-Caribou is an architecture for ASIC testing which uses Caribou hardware (ZCU102 + CaR board), driven by Spacely software. 
As a fully open-source system, Spacely-Caribou allows great flexibility across the entire stack. 

To use Spacely-Caribou to test your ASIC, you generally need to ensure that the hardware, firmware, and software are set up correctly. The flowchart below breaks this down into four "flows":

<p align="center">
<img src="https://github.com/SpacelyProject/spacely-docs/blob/main/figures/spacely-caribou/Spacely_Caribou_Flow.PNG" width="700">
</p>

Keep in mind that there are some dependencies between the flows. The arrows in the flowchart above give a suggestion for the order in which to work.

Here are the details on what you need to do:


## Caribou Hardware Setup Flow

In this flow, you set up the physical Caribou system (ZCU102 + CaR board), connect it to your test PC, and get it ready to operate.

1. [Create an OS Image on an SD Card](</spacely-caribou/hardware-setup/Creating an Operating System Image.md>)
2. [Set up the ZCU102](</spacely-caribou/hardware-setup/ZCU102 Setup.md>)

## Caribou Test Software Design Flow

In this flow, you set up the test software which will control your test stand. The test software consists of two parts: Peary (C++), running on your ZCU102, and Spacely (Python) running on your lab PC which is connected to the ZCU102. 

1. [Install Peary](</spacely-caribou/test-software-design/Installing and Running Peary.md>)
2. [Generate a Peary Memory Map](</spacely-caribou/test-firmware-design/Creating a Memory Map.md>) based on the memory map from your firmware design and add it to your  [Peary Device Files](</spacely-caribou/test-software-design/Peary Device Files.md>) .
3. [Build Peary](</spacely-caribou/test-software-design/Installing and Running Peary.md>) on the ZCU102.
4. [Write Spacely Routines](</spacely-caribou/test-software-design/Writing Routines for Spacely-Caribou.md>) to interact with Peary. If this is your first time using Spacely, also check out [Getting Started with Spacely](</fundamentals/Getting Started.md>) to interact with Peary.

## Caribou Test Firmware Design Flow 

In this flow, you will create a firmware design in Vivado, write a memory map for Peary/Spacely to use, synthesize your design, and flash it to the ZCU.

See: [Creating Firmware Designs for Spacely-Caribou](</spacely-caribou/test-firmware-design/Creating Firmware Designs for Spacely-Caribou.md>)

## Custom Firmware Design Flow (Optional)

Need to add your own firmware to provide a capability not provided by the Common Blocks repo? You can. 

Spacely provides an [autogeneration script](</spacely-caribou/test-firmware-design/Firmware Wrapper Autogeneration.md>) to quick and easily create a wrapper for your custom block which will allow it to be controlled over AXI just like any other Spacely-Caribou block. 

## Quick References

These documents may help you as you design your test stand.

[CaR Board Specifications](</spacely-caribou/reference/CaR Board Specifications.md>) -- Information on the I/O capabilities of the CaR board, including digital and analog. 

[Generic CaR Board Constraints](</spacely-caribou/reference/generic_CaR_board_constraints.xdc>) -- A starting point constraint file for your firmware design which contains the correct pin mapping.