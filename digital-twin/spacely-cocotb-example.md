#Spacely-Cocotb Example Project

The Spacely pytest suite includes a demo project for Spacely-Cocotb simulation, which is useful both as a test of the correctness Spacely-Cocotb framework, but also as a demo project to understand the various ways in which RTL can be co-simulated with Spacely. 

This demo project is co-packaged with Spacely (>v0.2.0). To get started, navigate to **spacely/PySpacely**, activate the Virtual Environment and run ```pytest -v -s```. This will run the built-in Spacely pytest suite (which should pass) and also copy a project named **pytest-golden** to your spacely-asic-config directory. 

**NOTE:** To use Spacely-Cocotb, you must have the executable for the Xcelium RTL simulator (xrun) on your path. If you do not, the Spacely-Cocotb related part of the test suite will fail. 

## Example Project Walkthrough 

### The Device Under Test
For simplicity, the device we wish to test in this project is a simple clocked adder, or **cadder**. The complete source code is given below:

```
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
Note the parameter, which can be used to introduce a logic error in this design, where the final carry bit of Z will be truncated. 

### Mode 0 (Plain Cocotb Simulation)

For our first and simplest test, we will test the cadder block at the block level, using traditional Cocotb test methodologies. For this purpose, we write the routine **ROUTINE_test_cadder_mode0**: 

```
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

Although it is easy to exhaustively verify such a small block, we use a constrained random verification strategy to demonstrate the capabilities of Cocotb. Our coverage model is defined as the cross between A and B inputs, with four bins for each. The *@Adder_Coverage* annotation before the sample_coverage() function instructs Cocotb to sample coverage every time this function is called, which is also when we compare the inputs to the output.

```
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

Note that this test is defined as **async**, takes "dut" as an argument, and uses asynchronous statements such as "await". This is the same as any other Cocotb test which interacts directly with the device under test. However, these will not remain the same in the digital twin tests that follow, as the test infrastructure becomes more sophisticated and abstract. 

### Mode 1 (Digital Twin Simulation with Firmware Blocks)


### Mode 2 (Digital Twin Simulation with a Complete Firmware Design)


<p align="center">
<img src="https://github.com/SpacelyProject/spacely-docs/blob/main/figures/digital-twin/pytest-golden-firmware.PNG" width="700">
</p>
