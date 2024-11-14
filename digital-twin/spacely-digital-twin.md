# Spacely Digital Twin 

A "digital twin" is a virtual copy of a physical system inside of a simulator. The digital twin is intended to mimic the physical system's response to stimuli, which allows you to conduct experiments on the digital twin instead of the real hardware. The benefits of this include:

1. Experimenting on the digital twin carries no risk of damaging real hardware.
2. The digital twin is often available before the physical hardware is manufactured.
3. The digital twin generally has greater observability (ability to probe internal signals) versus the physical hardware.

Using a digital twin is a type of simulation. However, it is distinct from typical RTL simulation, which aims to verify the internal functionality of a block in the most efficient way possible. Instead, the digital twin's goal is to mimic precisely the interface to the device under test (in this case, via Spacely-Caribou), even if this introduces inefficiencies. In this way, it comes much closer to replicating the results obtained in the lab, and can help prepare to obtain those results efficiently and accurately. 

# Creating an HDL Digital Twin 

The first step of Digital Twin simulation is creating the digital twin. This is an HDL system which mimics both the ASIC under test, and the FPGA firmware which will interact with that ASIC. 

First, make a new subfolder under spacely-asic-config/MyASIC/ called "hdl". In that folder, you will create three files:

- **CaribouDigitalTwinTop.sv** -- This is a SystemVerilog file which contains the top-level structure of your digital twin. It is written following some specific rules, which are described below. 
- **hdl_sources.txt** -- Each line of this text file is the path, relative or absolute, to HDL sources which are used for your Digital Twin. 
- **mem_map.txt** -- This is the same mem_map.txt used in the Spacely-Caribou flow for automatic memory map generation. 


## CaribouDigitalTwinTop.sv

This file should contain a module called CaribouDigitalTwinTop, with no ports. 

Inside the CaribouDigitalTwinTop module, instantiate your chip RTL, and also instantiate all of the AXI addressable blocks which will communicate with the chip and control it. Connect the signals between the AXI blocks and your chip using normal SystemVerilog syntax. 

The AXI interfaces will be automatically generated and connected to Cocotb for you. All you need to do is give Spacely a hint of where to connect these interfaces by leaving the comment ```/*AXI_INTERFACE(0x400000000)*/``` at the start of the port declarations for each AXI-connected block, where ```0x400000000``` is the memory address where this block resides in the overall firmware memory map -- this address needs to match with **mem_map.txt** in order for Spacely to correctly identify which AXI registers belong to which block.

### Example

Suppose we want to create a digital twin of a firmware image which contains only a *test_data_source* block from **spacely-caribou-common-blocks**. (In this case there is no ASIC, we are just testing the firmware by itself.) Let's assume that in **mem_map.txt**, the base address for test_data_source is 0x400005000. Then our CaribouDigitalTwinTop.sv file would look like this:

```
`timescale 1ns/1ps
module CaribouDigitalTwinTop();


   test_data_source_top tds ( /*AXI_INTERFACE(0x400005000)*/
			      .axi_clk(AXI_ACLK),
			      .axi_resetn(AXI_ARESETN));
   
endmodule // CaribouDigitalTwinTop
```


# Using Spacely to Interact with a Digital Twin 

With a few caveats, you can use the exact same Spacely routines to interact with a digital twin as you do to interact with the actual hardware. Here's what you need to do:

## MyASIC_Config.py File 

Set the following global variables:

```
USE_COCOTB = True
```
This switch will determine whether you interact with the real Caribou hardware, or with the digital twin. 
```
SIMULATOR  = "xcelium" 
```
Or your favorite simulator as appropriate.
```
HDL_TOP_MODULE = "CaribouDigitalTwinTop"
```
This variable should always be "CaribouDigitalTwinTop" when you are using the flow presented in this document. The only reason the variable exists is because it is possible to use Spacely for other Cocotb simulation which is not part of the Digital Twin flow. 


## MyASIC_Routines.py File 

In principle, no modification is required to Spacely routines to make them work with a Digital Twin. When you designate a routine to run through the main Spacely shell, Spacely will automatically make a copy of that routine and tweak its syntax to work with Cocotb.

The tweaks are:

1. Defining the routine function as async.
2. Placing the keyword "await" before every call to sg.INSTR\["car"\].get_memory or .set_memory.

### TODO Items to support:
- Simulator Timer calls
- Other Caribou functions. 
- Subroutines function?
