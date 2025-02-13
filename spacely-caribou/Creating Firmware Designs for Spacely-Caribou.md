# Creating Firmware Designs for Spacely-Caribou 

**Xilinx Vivado** is used to generate new firmware designs for Spacely-Caribou. Firmware designs are created by connecting together various **firmware blocks,** which may be either custom-designed blocks or reusable blocks from the **spacely-caribou-common-blocks** repo. 

If you are designing your own custom firmware blocks, refer to [Autogeneration Tools for Spacely-Caribou Firmware](</spacely-caribou/Autogeneration Tools for Spacely-Caribou Firmware.md>) for some tools to make your life easier. 

If you have your firmware blocks ready, this page will guide you through the process of creating a firmware design. 


## Create a New Vivado Project 

1. Select "File > Project > New..."
2. Following the steps in the Vivado New Project wizard, give your project a name, and choose its type to be "RTL Project."
3. On the "Default Part" screen, switch to "Boards" and search for "ZCU102".

<p align="center">
<img src="https://github.com/SpacelyProject/spacely-docs/blob/main/figures/spacely-caribou/CreatingFirmware_Fig1.PNG" width="700">
</p>


4. Finish the wizard and create your project. 
5. On the left sidebar, select "Create Block Design". Give the block design a name ending with "_bd". 
6. In the new block diagram, click the "+" button and search for "zynq" to add the Zynq UltraScale+ MPSoC.
7. Accept the prompt to run Block Automation for the Zynq IP.
8. Double-click on the Zynq IP to recustomize it:
	a. In the PS-PL Configuration tab, disable AXI HPM1 FPD, and set the bit width of AXI HPM0 FPD to 32 bits. This will be the main AXI interface used for your Spacely-Caribou design. 
9. Connect the maxihpm0_fpd_aclk pin to pl_clk0 to supply the AXI clock from pl_clk0. 
10. In the sources tab, right-click your new block diagram and choose "Create HDL Wrapper...". Let Vivado manage the wrapper and auto-update.
11. Right-click the new HDL wrapper and select "Set as Top"


## Add and Connect Your Blocks 

Now you can add all of the blocks that you wish to use to your Vivado block design. 

To include a block from **spacely-caribou-common-blocks**:
1. Click the "+" button in the Sources tab 
2. Choose "Add or create design sources" 
3. Choose "Add Files"
4. Navigate to the spacely-caribou-common-blocks repository, enter the sub-folder for the block you wish to instantiate, and add all the necessary source files. 
5. Wait for the file hierarchy to update, and then drag and drop the top-level file from that block into your block design. 

If a block follows the standard spacely-caribou-common-blocks convention, you will at minimum need to include three files: **BlockName_top.v**, **BlockName_interface.sv**, and **src/BlockName.sv**. However, some blocks may have additional sub-modules or constraints that you need to include. Be sure to read and understand the README for each block you plan to use, which will explain which files must be included. 

Almost all common blocks will require including the AXI4-Lite interface from **spacely-caribou-common-blocks/axi4lite_interface** in order to use them.

To include a Xilinx IP block:
1. Right-click anywhere in the block diagram. 
2. Search for the IP block you wish to use and click on its name to instantiate it. 

After adding AXI-enabled blocks, you will be prompted to run Connection Automation, which you should accept. This will easily add in the AXI crossbar and other infrastructure for communicating with your blocks over AXI. 

Here is an example design after adding two common blocks, prior to running Connection Automation:

<p align="center">
<img src="https://github.com/SpacelyProject/spacely-docs/blob/main/figures/spacely-caribou/CreatingFirmware_Fig2.PNG" width="700">
</p>

The same design after running Connection Automation: 

<p align="center">
<img src="https://github.com/SpacelyProject/spacely-docs/blob/main/figures/spacely-caribou/CreatingFirmware_Fig3.PNG" width="700">
</p>

### Allocating Address Space

Each AXI-enabled block that you add will receive an address in the AXI address space. If you are using the default Spacely-Caribou OS image, only memory space starting from 0x4_0000_0000 is mapped for AXI blocks, so your lowest address block should have a base address of 0x4_0000_0000. Most blocks consume only a very small amount of address space, but for simplicity you can give each block 4K of address space, resulting in your second block being placed at 0x4_0000_1000, as shown below:

<p align="center">
<img src="https://github.com/SpacelyProject/spacely-docs/blob/main/figures/spacely-caribou/CreatingFirmware_Fig4.PNG" width="700">
</p>


## Assign Pins 
In order to connect your firmware design to your ASIC, you need to assign external pins. CMOS_IN and CMOS_OUT signals on the CaR board are driven from the ZCU102 via a differential (LVDS) channel, so you should first use the Xilinx Utility Buffer IP to convert any single-ended signal in your firmware design which is intended to connect to a CMOS_IN/CMOS_OUT channel into a differential signal. You can connect this signal to an pin simply by right-clicking the differential port on the utility buffer and selected "Make External":

<p align="center">
<img src="https://github.com/SpacelyProject/spacely-docs/blob/main/figures/spacely-caribou/CreatingFirmware_Fig5.PNG" width="700">
</p>

After creating these ports, you will need to write a constraint file which assigns them to the correct FPGA pins to be connected to CMOS_IN/CMOS_OUT channels on the CaR board. (TBA)

## Synthesize, Implement, and Close Timing
TBA

### Optional: Digital Twin Simulation with Spacely 
If you choose, you can simulate a "digital twin" of your entire firmware design using Spacely. You can export your firmware netlist with the command ``` write_verilog -mode funcsim <sim_netlist.v>``` and follow the instructions in [Spacely Digital Twin](</digital-twin/spacely-digital-twin.md>)

## Generate a Bitstream 
TBA