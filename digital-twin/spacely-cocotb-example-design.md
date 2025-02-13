# Spacely-Cocotb Example Design

The Spacely pytest suite includes a demo project for Spacely-Cocotb simulation, which is used as a test of the correctness Spacely-Cocotb framework, but is also useful as a demo project to understand the various ways in which RTL can be co-simulated with Spacely. 

This demo project is included in the Spacely repository (>=v0.2.0). To get started, navigate to **spacely/PySpacely**, activate the Virtual Environment and run ```pytest -v -s```. This will run the built-in Spacely pytest suite and also copy a project named **pytest-golden** to your spacely-asic-config directory. 

**IMPORTANT NOTE:** In order for all of the Spacely-Cocotb tests to pass, some user customization which is specific to your work area is required. In particular, you must (1) place the Xcelium executable "xrun" on your path, and (2) provide links to certain generic RTL repositories in the **pytest-golden** project. The first time you attempt to run pytest, it will fail and throw "USER CUSTOMIZATION NEEDED" warnings which will instruct you in what you need to provide. 

## Example Project Walkthrough 

### The Device Under Test
For simplicity, the device we wish to test in this project is a simple clocked adder, or **cadder**. The complete source code is given below:

```systemverilog
module cadder #(CARRY_ERROR=0) (
				input logic [3:0]  A,
				input logic [3:0]  B,
				input logic 	   clk,
				output logic [4:0] Z);

   logic [3:0] 					   Z_4b;

   assign Z_4b = A + B;

   always @(posedge clk) begin
      if(CARRY_ERROR)
	Z <= Z_4b;
      else
	Z <= A + B;
   end
   
endmodule // cadder
```

On a positive clock edge, the cadder samples its 4-bit inputs A and B, and asserts the sum on its 5-bit output Z. 
Note the CARRY_ERROR parameter, which can be used to introduce a logic error in this design, where the final carry bit of Z will be truncated. 

### Mode 0 (Simple Cocotb Simulation)

For our first and simplest test, we will test the cadder block at the block level, using traditional Cocotb test methodologies. 

For this test mode, we set the variables:
```python
sg.USE_COCOTB=True
sg.SIMULATOR="xcelium"
sg.HDL_TOP_MODULE="cadder"
sg.HDL_SOURCES="hdl_sources_mode_0.txt"
sg.COCOTB_BUILD_ARGS=[]
sg.TWIN_MODE = 0
```

To test the block, we write the Spacely test routine **ROUTINE_test_cadder_mode0**: 

```python
#Calling this function samples coverage and also checks correctness of outputs.
@Adder_Coverage
def sample_coverage(A,B,Z):
    assert A+B == Z

async def ROUTINE_test_cadder_mode0(dut):
    """Test cadder block at block-level (twin mode 0)"""

    cocotb.fork(Clock(dut.clk, 1000).start())

    await RisingEdge(dut.clk)
    await Timer(1, units="ns")

    for _ in range(100):
        A_val = random.randint(0,15)
        B_val = random.randint(0,15)

        dut.A.value = A_val
        dut.B.value = B_val

        await RisingEdge(dut.clk)
        await Timer(1, units="ns")
        sample_coverage(A_val,B_val, dut.Z.value.integer)

    cover_percentage = coverage_db["dut"].cover_percentage
    sg.log.info(f"Coverage Percentage acheived in this test: {cover_percentage} %")

    #With 16 cover bins and 100 samples, there's roughly a 1 in a million chance of
    #this assertion failing by random chance.
    assert cover_percentage > 0.9
```

We use cocotb.fork() to start a coroutine to generate a clock. We then supply values to *A* and *B*, wait for a clock edge, and compare the sum to the value on *Z*. 

Note that the function **ROUTINE_test_cadder_mode0** is defined as **async**, takes "dut" as an argument, and uses asynchronous statements such as "await". This is the same as any other Cocotb test which interacts directly with the device under test. However, this is different from the digital twin tests that follow; as the test infrastructure becomes more sophisticated and abstract, we will no longer use low-level asynchronous programming statements.

Although it is easy to exhaustively verify such a small block, we use a constrained random verification strategy to demonstrate the capabilities of Cocotb. Our coverage model is defined as the cross between A and B inputs, with four bins for each. The *@Adder_Coverage* annotation before the sample_coverage() function instructs Cocotb to sample coverage every time this function is called, which is also when we compare the inputs to the output.

Here is the definition of Adder_Coverage:

```python
#Relation defines what it means for a value to fall inside a bin (low,high)
range_relation = lambda val_, bin_ : bin_[0] <= val_ <= bin_[1]

#Define a set of CoverPoints that cover combinations of high and low values in the adder.
Adder_Coverage = coverage_section (
    CoverPoint("dut.A",
               vname="A",
               rel = range_relation,
               bins = [(0,3),(4,7),(8,11),(12,15)]),
    CoverPoint("dut.B",
               vname="B",
               rel = range_relation,
               bins = [(0,3),(4,7),(8,11),(12,15)]),
    CoverCross("dut.A_vs_B",
        items=["dut.A","dut.B"]))
```

In each Spacely-Cocotb test we run, we need to create an hdl_sources file which lists the sources that should be used for the test.  

For this test, the file *hdl_sources_mode_0.txt* just contains a reference to the DUT RTL itself:

```
// -- DUT Source --
cadder.sv
```

### Mode 1 (Digital Twin Simulation with Firmware Blocks)

Once we have convinced ourselves that the block design of the cadder is correct, we must now begin designing the firmware which we will use to verify the cadder once it is implemented on an actual chip. 

For this test mode, we set the variables:
```python
sg.USE_COCOTB=True
sg.SIMULATOR="xcelium"
sg.HDL_TOP_MODULE="cadder_twin_mode1"
sg.HDL_SOURCES="hdl_sources_mode_1.txt"
sg.COCOTB_BUILD_ARGS=[]
sg.TWIN_MODE = 1
```

To make our task easier, we will leverage the generic firmware in the **spacely-caribou-common-blocks** repository. We will use a **logic_clk_div** block to generate a variable-speed clock for the ASIC, and we will use an **Arbitrary_Pattern_Generator** block to generate all possible combinations of inputs for *A* and *B*, and also to read back the value of *C* which is generated by the ASIC so that we can check its correctness. If you are unfamiliar with these blocks, you can review their README files in https://github.com/SpacelyProject/spacely-caribou-common-blocks. 

We write a simple RTL netlist which connects these two firmware blocks to the DUT:

```systemverilog
module cadder_twin_mode1();

   logic dut_clk;
   logic [3:0] A, B;
   logic [4:0] Z;
         
   //cadder (Device Under Test)
   cadder  #(.CARRY_ERROR(0)) uDUT (.clk(dut_clk),.A(A),.B(B),.Z(Z));   

   //Test Firmware
   logic_clk_div_top logic_clk_div_top_0 (/*AXI_INTERFACE(0x400001000)*/
				  .master_clk(AXI_ACLK),
				  .output_clk(dut_clk),
				  .axi_resetn(AXI_ARESETN),
				  .axi_clk(AXI_ACLK));
   
   Arbitrary_Pattern_Generator_top #(.NUM_SIG(8), 
				     .NUM_SAMP(300)
				     ) test_apg (/*AXI_INTERFACE(0x400000000)*/
						.axi_clk(AXI_ACLK),
						.axi_resetn(AXI_ARESETN),
						.wave_clk(dut_clk),
						.input_signals({3'b0,Z}),
						.output_signals({A,B}));
   
endmodule // cadder_fw_mode1
```

Note the special ```/*AXI_INTERFACE()*/``` comments which are placed at the beginning of the port list for each of the firmware modules. These comments are used by Spacely-Caribou to identify which AXI registers belong to which firmware block. 

The AXI registers are defined in the **mem_map.txt** file, which is also in the /hdl/ directory: 

```
// Memory map for cadder test firmware

*BASE 0x400001000 //logic_clk_div_top_0
divider_cycles,0x0,0xffffffff,True,True
divider_rstn,0x4,0x1,True,True

*BASE 0x400000000 //Arbitrary_Pattern_Ge_0
run,0x0,0x1,False,True
clear,0x4,0x1,False,True
write_channel,0x8,0xffffffff,True,True
read_channel,0xc,0xffffffff,True,False
write_defaults,0x10,0xffffffff,True,True
async_read_channel,0x14,0xffffffff,True,False
sample_count,0x18,0xffffffff,True,False
n_samples,0x1c,0xffffffff,True,True
control,0x20,0xff,True,True
write_buffer_len,0x24,0xffffffff,True,False
next_read_sample,0x28,0xffffffff,True,False
wave_ptr,0x2c,0xffffffff,True,False
status,0x30,0x7,True,False
dbg_error,0x34,0xffffffff,True,False
param_NUM_SIG,0x38,0xffffffff,True,False
param_NUM_SAMP,0x3c,0xffffffff,True,False
```

This **mem_map.txt** file is exactly identical to the memory map file that must be created in order to build the actual test firmware, so creating it for Digital Twin test is no additional burden. The format of this file is explained in [Autogeneration Tools for Spacely-Caribou Firmware](</spacely-caribou/Autogeneration Tools for Spacely-Caribou Firmware.md>).

To interact with our design via the test firmware, we will write a new routine called **ROUTINE_test_cadder_mode1_2**. *This exact same Spacely routine will work for Twin Mode 1, Twin Mode 2, and even testing the actual hardware!*

In the first few lines of the routine, we set up variables and set up the logic_clk_div block to send a clock to the DUT which is (arbitrarily) 1/6th the AXI clock speed. Note how we are interacting with the firmware using the **sg.INSTR["car"]** construct, which is exactly the same way we interact with an actual Caribou system. Because we have set TWIN_MODE=2, Spacely knows to redirect our commands instead to the emulated firmware AXI bus. 
```python
def ROUTINE_test_cadder_mode1_2():
    """Test cadder block include a digital twin of test firmware (twin mode 1 or 2)"""

    sg.INSTR["car"].debug_memory = True
    
    # We expect the output to be offset by 2 cycles from the input: one cycle for the AWG to
    # assert the input value, and one cycle for the cadder to clock that value. 
    LOOPBACK_OFFSET_CYC = 2
    
    sg.INSTR["car"].set_memory("divider_cycles",5)
    sg.INSTR["car"].set_memory("divider_rstn",0)
    sg.INSTR["car"].set_memory("divider_rstn",1)
```

Next, we algorithmically create a stimulus vector for the Arbitrary Pattern Generator which will exercise all combinations of *A* and *B*. Recall that we connected *A* (4 bits) and *B* (4 bits) to the 8-bit output bus of the Arbitrary Pattern Generator, so if we simply cycle through all 256 possible values of this 8-bit bus, this will cover all combinations of A and B. 

```python
    sg.INSTR["car"].set_memory("clear",1)

    #Try all possible combinations of the 4-bit (A) + 4-bit (B) inputs.
    input_vector = [n for n in range(256)]
    
    for vec in input_vector:
        sg.INSTR["car"].set_memory("write_channel", vec)

    sg.INSTR["car"].set_memory("n_samples",256 + LOOPBACK_OFFSET_CYC)
```

In the next few lines, we tell the APG to run, which will pass our test values to the DUT and capture the results. We wait for it to finish, then retrieve our results over AXI. Recall that the APG and the DUT receive the same clock from logic_clk_div, so one result is processed every clock cycle. 
Note the function call below to ```sg.INSTR["car"].dly_min_axi_clk(50)``` -- this function will result in a delay of 50 AXI clock cycles *no matter whether you are emulating the firmware or running on actual hardware.* Using this function for delays (instead of time.sleep() or await statements) makes your test routine completely portable between digital twin test and actual hardware test.

```python
    sg.INSTR["car"].set_memory("run",1)

    while True:
        status = sg.INSTR["car"].get_memory("status")
        if status == 0:
            break
        else:
            sg.log.debug("<test> Waiting for APG idle")
            sg.INSTR["car"].dly_min_axi_clk(50)


    result_vector = []
    sg.log.debug("<test> Starting to read back results.")
    for n in range(256 + LOOPBACK_OFFSET_CYC):
        result_vector.append(sg.INSTR["car"].get_memory("read_channel"))

    sg.log.debug("<test> Finished reading back results.")
```

Finally, we print the results to the screen and use assert statements to check the correctness of the block:

```python
    for i in range(256):
        input_A = input_vector[i] & 0xf
        input_B = input_vector[i] >> 4
        output  = result_vector[i+LOOPBACK_OFFSET_CYC]

        print(f"{input_A:2} + {input_B:2} = {output:2}   ",end='')

        assert input_A + input_B == output
        
        if i%8 == 0:
            print("") #Newline after every 8 
        
```

Finally, our sources file is *hdl_sources_mode_1.txt*, shown below. This file references the DUT (cadder.sv), the top level which connects firmware to DUT (cadder_twin_mode1.sv), and the common blocks structures we used:

```
// -- Top-level files for Digital Twin Simulation --
cadder_twin_mode1.sv

cadder.sv

SOURCES hdl_sources_common_blocks.txt

// -- Common Blocks used in MODE 1 --
$COMMON_BLOCKS/Arbitrary_Pattern_Generator/src/Arbitrary_Pattern_Generator.sv
$COMMON_BLOCKS/Arbitrary_Pattern_Generator/Arbitrary_Pattern_Generator_interface.sv
$COMMON_BLOCKS/Arbitrary_Pattern_Generator/Arbitrary_Pattern_Generator_top.v

$COMMON_BLOCKS/axi4lite_interface/axi4lite_slave_interface.sv
$COMMON_BLOCKS/axi4lite_interface/axi4lite_interface_top.sv
$COMMON_BLOCKS/axi4lite_interface/mem_regs.sv

$COMMON_BLOCKS/logic_clk_div/logic_clk_div_interface.sv
$COMMON_BLOCKS/logic_clk_div/logic_clk_div_top.v
$COMMON_BLOCKS/logic_clk_div/src/logic_clk_div.sv

$COMMON_BLOCKS/Xilinx_Blocks/Xilinx-CDC-Structure-Sim-Models.sv
```

The line ```SOURCES hdl_sources_common_blocks.txt``` instructs Spacely that it should also read the file *hdl_sources_common_blocks.txt* to find more sources. In this case, that file simply defines the macro $COMMON_BLOCKS, which is a pointer to where you have downloaded the **spacely-caribou-common-blocks** repository. Because Spacely cannot predict where you have placed this repo, the macro definition is initially "???". You must replace the "???" with ```/path/to/spacely-caribou-common-blocks``` in order for the pytest suite to pass:

```
// -- Macros --
// COMMON_BLOCKS - should point to the spacely-caribou-common-blocks repo, found at
//                 https://github.com/SpacelyProject/spacely-caribou-common-blocks
DEF COMMON_BLOCKS ???
```

### Mode 2 (Digital Twin Simulation with a Complete Firmware Design)

Finally, let's suppose that we have created our firmware design in Vivado and we are about to start testing with our chip. Can we go back to digital twin simulation and confirm that our firmware *actually* works with the ASIC the way we expect it to? You bet we can! 

For this test mode, we define the variables:

```python
sg.USE_COCOTB=True
sg.SIMULATOR="xcelium"
sg.HDL_TOP_MODULE="cadder_twin_mode2"
sg.HDL_SOURCES="hdl_sources_mode_2.txt"
sg.FW_TOP_MODULE="pytest_golden_fw_bd"
sg.COCOTB_BUILD_ARGS=["-top glbl"]
sg.TWIN_MODE=2
```

In Vivado, our firmware design looks like this: 

<p align="center">
<img src="https://github.com/SpacelyProject/spacely-docs/blob/main/figures/digital-twin/pytest-golden-firmware.PNG" width="900">
</p>

Let's break down the blocks that we see here:
- The **logic_clk_div** and **Arbitrary_Pattern_Generator** blocks play the same role as in the previous test. 
- The **Zynq UltraScale+ MPSoC** represents the ZCU102 SoC; this block gets placed at the beginning of [any Spacely-Caribou firmware design.](</spacely-caribou/Creating Firmware Designs for Spacely-Caribou.md>)
- The **AXI Interconnect** and **Processor System Reset** blocks are automatically created and routed by Vivado when you choose "Connection Assistance". 
- The **Slice** and **Concat** blocks perform basic vector operations, extracting or combining individual bits of a vector. The **Constant** block assigns the two unused bits of the APG input to zero. 
- The **Utility Buffer** blocks convert differential I/O to single-ended signals. This is needed for a real Caribou firmware design because all of the CMOS_IN/CMOS_OUT channels of the CaR board are controlled by LVDS signals from the FPGA. 

Once we create and synthesize this design, we can export its netlist using the TCL command ```write_verilog -mode funcsim <sim_netlist.v>```

This exported netlist is saved in the /hdl/ folder as **pytest_golden_fw_bd.v**.

Now that we have an actual firmware design, we write a simple wrapper to connect the firmware to the DUT in *cadder_twin_mode2.sv*:

```systemverilog
`timescale 1ns/1ps
module cadder_twin_mode2();

   logic [3:0] A, B;
   logic [4:0] Z, Z_b;
   logic       dut_clk;
   
   //cadder (Device Under Test)
   cadder  #(.CARRY_ERROR(0)
	     ) uDUT (.clk(dut_clk),
		     .A(A),
		     .B(B),
		     .Z(Z));

   assign Z_b = ~Z;

   //Test Firmware
   pytest_golden_fw_bd uFW (/*AXI_PASSTHROUGH(2)*/
			    .A_0_clk_n(),.A_0_clk_p(A[0]),
			    .A_1_clk_n(),.A_1_clk_p(A[1]),
			    .A_2_clk_n(),.A_2_clk_p(A[2]),
			    .A_3_clk_n(),.A_3_clk_p(A[3]),
			    .B_0_clk_n(),.B_0_clk_p(B[0]),
		            .B_1_clk_n(),.B_1_clk_p(B[1]),
			    .B_2_clk_n(),.B_2_clk_p(B[2]),
			    .B_3_clk_n(),.B_3_clk_p(B[3]),
			    .Z_0_clk_n(Z_b[0]),.Z_0_clk_p(Z[0]),
			    .Z_1_clk_n(Z_b[1]),.Z_1_clk_p(Z[1]),
			    .Z_2_clk_n(Z_b[2]),.Z_2_clk_p(Z[2]),
			    .Z_3_clk_n(Z_b[3]),.Z_3_clk_p(Z[3]),
			    .Z_4_clk_n(Z_b[4]),.Z_4_clk_p(Z[4]),
			    .dut_clk_clk_n(),.dut_clk_clk_p(dut_clk));

endmodule // cadder_twin_mode2
```

Note the special comment ```/*AXI_PASSTHROUGH(2)*/``` which is placed at the beginning of the firmware port list. This comment gives Spacely a hint that there are two AXI-enabled blocks (logic_clk_div and Arbitrary_Pattern_Generator) inside the firmware which it will need to connect. 

Note also the naming convention of the firmware ports, which is determined by Vivado (for differential signals, "_clk_p" and "_clk_n" are appended to the positive and negative versions of the ports. For simplicity, we use single-ended signals in the wrapper. For firmware outputs, we simply take the "_clk_p" signal and leave the "_clk_n" unconnected. For firmware inputs, we generate the complement of the signal in the wrapper. 

The Spacely routine we use to test this design, and the mem_map.txt file, are both identical to Mode 1. 

For this mode, we use *hdl_sources_mode_2.txt*, which contains the wrapper (cadder_twin_mode2.sv) and the DUT (cadder_timescale.sv). Our firmware design is automatically included without adding it here because of the value we set for sg.FW_TOP_MODULE. 

```
// -- Top-level file for Digital Twin Simulation -- 
cadder_twin_mode2.sv

cadder_timescale.sv

SOURCES hdl_sources_unisim.txt

// -- Unisim Simulation Models --
$UNISIM/verilog/src/glbl.v
$UNISIM/verilog/src/unisims/*.v
```

The synthesized Vivado netlist for the firmware relies on Vivado HDL primitives, which are provided by the Xilinx Unisim Library. This is another library that the user must provide a path to, specifically by editing the file *hdl_sources_unisim.txt*:

```
// -- Macros --
// NOTE: To carry out digital twin simulation, you MUST define these macros.
// UNISIM - should point to the Xilinx Unisim models library, found at
//          https://github.com/Xilinx/XilinxUnisimLibrary
DEF UNISIM ???
```

**NOTE:** Things to watch out for when using the Xilinx Unisim Library:
- Timescale: The Xilinx Unisim primitives have timescales defined in their source files. According to IEEE verilog simulation standards, if one file in a simulation has a timescale defined, then all files must have a timescale defined. Note that *cadder_twin_mode2.sv* defines a timescale on the first line. And note that we have replaced *cadder.sv* in the sources file with *cadder_timescale.sv* which is identical except that it provides a timescale on the first line. 
- Glbl file: The file glbl.v is part of the Xilinx Unisim Library, and contains some necessary global variables for the HDL primitives. As such simulation will only work if this file is set as a "top" file in addition to your actual top HDL file. This is the reason why we add the string "-top glbl" to sg.COCOTB_BUILD_ARGS

## Expected results

### Normal 
If you have installed Spacely correctly **and done the User Customization steps correctly**, you should expect to see the following results when running ```pytest -v```:
```
test/Spacely_Cocotb_test.py::test_cadder_mode0 PASSED 
test/Spacely_Cocotb_test.py::test_cadder_mode1 PASSED 
test/Spacely_Cocotb_test.py::test_cadder_mode2 PASSED
```

### Introducing an error
Try editing the module definition or instantiation of cadder in **spacely-asic-config/pytest_golden/hdl** to set CARRY_ERROR = 1. Assuming that you inject an error into all three test modes, you should now see the following:
```
FAILED test/Spacely_Cocotb_test.py::test_cadder_mode0 - SystemExit: ERROR: Failed 1 of 1 tests.
FAILED test/Spacely_Cocotb_test.py::test_cadder_mode1 - SystemExit: ERROR: Failed 1 of 1 tests.
FAILED test/Spacely_Cocotb_test.py::test_cadder_mode2 - SystemExit: ERROR: Failed 1 of 1 tests.
```
If you run ```pytest -v -s```, you can see the actual failing result for each mode. 

Twin Mode 1 and 2 should fail with ```AssertionError: assert (15 + 1) == 0``` because these modes iterate through all possible values of A and B, and 15 + 1 is the first test case that will fail in the presence of a Carry Error. 

Because Twin Mode 0 uses constrained random verification, the failing test case may vary, but because of the nature of the Carry Error, you should observe that the actual result is smaller than the expected result by 16, for example ```AssertionError: assert (12 + 7) == 3```

Additionally, Spacely should notice the files that you modified to inject the error, and a warning similar to the following should appear somewhere in your pytest log:
```
test/Spacely_Cocotb_test.py::test_cadder_mode1 <WARN> <2025-02-13 09:50:03> The following files have been MODIFIED from their golden reference versions. The pytest suite may not return the correct results: ['spacely-asic-config/pytest_golden/hdl/cadder_timescale.sv', 'spacely-asic-config/pytest_golden/hdl/cadder.sv']
```