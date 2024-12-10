# Spacely Digital Twin 

A "digital twin" is a virtual copy of a physical system inside of a simulator. The digital twin is intended to mimic the physical system's response to stimuli, which allows you to conduct experiments on the digital twin instead of the real hardware. The benefits of this include:

1. Experimenting on the digital twin carries no risk of damaging real hardware.
2. The digital twin is often available before the physical hardware is manufactured.
3. The digital twin generally has greater observability (ability to probe internal signals) versus the physical hardware.

Using a digital twin is a type of simulation. However, it is distinct from typical RTL simulation, which aims to verify the internal functionality of a block in the most efficient way possible. Instead, the digital twin's goal is to mimic precisely the interface to the device under test (in this case, via Spacely-Caribou), even if this introduces inefficiencies. In this way, it comes much closer to replicating the results obtained in the lab, and can help prepare to obtain those results efficiently and accurately. 



# Config Options for Spacely Digital Twin Simulation 

To run a digital twin variation, you first need to set some important variables in your *MyASIC_Config.py* file. 

```
USE_COCOTB = True
```
This switch must be "True" to do any RTL simulation with Spacely (including digital twin).
If you are using a Caribou system, setting this switch will route all Caribou methods to the digital twin instead of the actual hardware.
```
SIMULATOR  = "xcelium" 
```
Xcelium is the recommended and supported simulator, but you can try using others. See "External Environment Variables" below.

```
COCOTB_BUILD_ARGS = []
```
Pass any needed additional build arguments as a list to Cocotb. 

```
HDL_TOP_MODULE = "myModule"
```
This is the name of your HDL top module, which includes everything you want to simulate (potentially including both an ASIC and your test firmware). 
Later, we will create a file called */spacely-asic-config/MyASIC/hdl/myModule.sv*
```
TWIN_MODE = (0 or 1 or 2)
```
This variable allows you to select from three possible HDL simulation flows. See below for details...

## TWIN_MODE = 0
Setting TWIN_MODE to 0 allows you to run regular Cocotb simulations with Spacely, which do not involve Caribou or any digital twins. 
Your test routines should accept an argument called "dut" which will be set to reference *myModule*. 

## TWIN_MODE = 1
Setting TWIN_MODE to 1 is useful for early digital twin simulations, when you may just have a few custom firmware blocks but have not started to work in Vivado yet. 
In this mode, you will instantiate your firmware blocks directly in *myModule*, and leave specially-formatted comments to tell Spacely where to hook up AXI interfaces to 
allow your testbench to talk to these blocks. 

## TWIN_MODE = 2
Setting TWIN_MODE to 2 will instruct Spacely to take your firmware design directly from a netlist written by Vivado, and pre-process it to automatically create a digital twin. 
When you use this mode, you should also set:
```
FW_TOP_MODULE = "myFwModule"
```
Where "myFwModule" is the name of your top-level firmware netlist exported from Vivado. This netlist should be saved as: */spacely-asic-config/MyASIC/hdl/myFwModule.v*

## External Environment Variables

In order to run RTL simulations, you also need to have the correct simulator environment variables. For example, if you want to run Xcelium, you will need the *xrun* executable on your path.

**NOTE:** If you want to run simulations with Xilinx SecureIP, you will also need to set the environment variable ```XILINX_VIVADO``` to your Vivado install directory. This is necessary to use the SecureIP cell lists provided by Vivado. Alternatively, you can just reference the exact secureIP that you need in **hdl_sources.txt**.


# Creating an HDL Digital Twin 

After setting the correct configuration options, the first step of Digital Twin simulation is creating the digital twin. This is an HDL system which mimics both the ASIC under test, *and* the FPGA firmware which will interact with that ASIC. 

First, make a new subfolder under spacely-asic-config/MyASIC/ called "hdl". In that folder, you will create three files:

## 1. Digital Twin Top level
Create a file called **myModule.sv**, using the same module name as you wrote in the ```HDL_TOP_MODULE``` variable. 

Inside myModule, instantiate your chip RTL, and also instantiate your firmware blocks, following the instructions below for TWIN_MODE 1 or 2. Connect the signals between the AXI blocks and your chip using normal SystemVerilog syntax -- only the AXI signals are handled specially. 

### Instantiating Firmware Blocks (TWIN_MODE = 1)
In this mode, you should instantiate individual firmware blocks as SystemVerilog blocks. But do not connect their AXI interfaces: the AXI interfaces will be automatically generated and connected to Cocotb for you. All you need to do is give Spacely a hint of where to connect these interfaces by leaving the comment ```/*AXI_INTERFACE(0x400000000)*/``` at the start of the port declarations for each AXI-connected block, where ```0x400000000``` is the memory address where this block resides in the overall firmware memory map -- this address needs to match with **mem_map.txt** in order for Spacely to correctly identify which AXI registers belong to which block.

If any of your blocks take the global AXI clock or reset as an input, they can be connected to *AXI_ACLK* and *AXI_ARESETN* respectively. There's no need to define these signals, as they will be defined automatically by Spacely when it sets up the AXI interfaces.

**Example:**
Suppose we want to create a digital twin of a firmware image which contains only a *test_data_source* block from **spacely-caribou-common-blocks**. (In this case there is no ASIC, we are just testing the firmware by itself.) Let's assume that in **mem_map.txt**, the base address for test_data_source is 0x400005000. Then our top level file would look like this:

```
`timescale 1ns/1ps
module CaribouDigitalTwinTop();


   test_data_source_top tds ( /*AXI_INTERFACE(0x400005000)*/
			      .axi_clk(AXI_ACLK),
			      .axi_resetn(AXI_ARESETN));
   
endmodule // CaribouDigitalTwinTop
```

### Instantiating Firmware Blocks (TWIN_MODE = 2)
In this mode, you can include firmware directly from a Vivado-exported netlist. First, run the following command to export your netlist from Vivado.
```
write_verilog -mode funcsim <sim_netlist.v>
```
You should choose the netlist name so that it is the same as that of the top firmware module, which should also be the same as ```FW_TOP_MODULE```.
Copy this file into your /hdl/ directory, and instantiate it as normal, connecting and I/Os to your ASIC. Finally, leave the special comment ```/*AXI_PASSTHROUGH(N)*/``` at the beginning of the port connections of this module, where "N" is the total number of AXI-addressable blocks in your firmware. 


**Example:**
Suppose that our firmware top level is called "SP3_Firmware_bd". (The "_bd" stands for "Block Design", from Vivado). Suppose also that we have a total of seven AXI-addressable firmware blocks in this image. Then the instantiation inside the top level HDL file may look like this: 
```
SP3_Firmware_bd uFW (/*AXI_PASSTHROUGH(7)*/
			.DTP0(DTP0),
			.DTP1(DTP1),
			.DTP2(DTP2)...);
```
 

## 2. HDL Sources
Create a file in the /hdl/ directory named **hdl_sources.txt**. Each line of this text file should contain the absolute path to HDL sources which are needed for your simulation. 
In order to make this less tedious, you can define macros with the "DEF" keyword. Commented lines which begin with "//" are also allowed. 
For example, suppose we want to include these two HDL files: /asic/projects/myProject/module_1.sv and /asic/projects/myProject/module_2.sv. An example hdl_sources.txt file would look like this:

```
DEF MYPROJECT /asic/projects/myProject/module_1

// -- Modules for myProject --
$MYPROJECT/module_1.sv
$MYPROJECT/module_2.sv
```

## 3. mem_map.txt
Create a file in the /hdl/ directory named **mem_map.txt**. This is the same mem_map.txt used in the Spacely-Caribou flow for automatic memory map generation. However, there is one additional requirement: Each BASE address line must be followed with a comment that gives the name of the instance in your top-level firmware design which uses that address range. This helps Spacely associate memory addresses to blocks. 

For example:
```
*BASE 0x400000000 //logic_clk_div_top_0
```


# Using Spacely to Interact with a Digital Twin 

In general, you can use the exact same Spacely routines to interact with a digital twin as you do to interact with the actual hardware. When the ```USE_COCOTB``` flag is set, any Caribou commands, for example SpacelyCaribou.set_memory(), will be sent to the digital twin instead of actual hardware. 

## Creating Delays

Spacely digital twin simulation is not limited by the latency of the Caribout-to-Host-PC interface. This means that if your routines implicitly rely on this delay, they may work differently in digital twin simulation. 

In principle, the best approach is to not rely on specific interface latencies. However, an easy way to fix this is to use the function **SpacelyCaribou.dly_min_axi_clk(N)**, which inserts a delay of N axi clock cycles. This works with both real and twin hardware: with real hardware, it is evaluated to a time.sleep() statement, while for digital twin hardware, it is evaluated to a cocotb.triggers.ClockCycles() statement. 


# Digital Twin Creation Flow

**Note:** This section contains technical information which you do not need to understand unless you plan to develop for Spacely-Cocotb.

Cocotb tests are run in a separate environment which does not share a global namespace with the main Spacely shell. 

First, these steps occur in the Spacely environment:
1. **Gather HDL Sources:** hdl_sources.txt is parsed to obtain a list of source files. 
2. **Create Digital Twin HDL:** CaribouDigitalTwinTop.sv is parsed, and the additional AXI interface connections are added automatically to create a fully-functional digital twin. A comment is added to the top of the file mapping the AXI interfaces to their corresponding memory location, which will later be picked up in the Cocotb environment.
3. **Create Testbench:** The testbench function is copied from MyASIC_Routines.py with the modifications described above. 
4. **Run Test:** The simulator is initialized, source files are compiled, and the test is run. 

Next, in the Cocotb environment, the following steps occur in the entry function before handing over control to the user routine:
1. **sg.log setup:** A new sg.log instance is created inside the Cocotb environment to allow logging to function normally.
2. **CaribouTwin Initialization:** A CaribouTwin object is created. This object examines the mem_map.txt file and the digital twin HDL created in the Spacely environment, and uses them to create software AXI interfaces which are linked to the digital twin.
3. **AXI Clock/Reset Init:** AXI_ACLK is initialized at a standard speed of 100 MHz, and an AXI reset is pulsed to initialize the DUT into a known state.

## Assigning AXI block addresses and numbers:

The function axi_block_info_from_mem_map() parses the mem_map.txt file and returns two dictionaries:
- axi_block_addr maps block names to hex addresses
- axi_block_num  maps block names to AXI interface nums.

These two dictionaries are used in conjuction to insert new AXI interfaces with consistent numbering into the user's top module. When the user calls set_memory() or get_memory(), these dictionaries are used to look up the appropriate AXI interface to handle each memory operation. 