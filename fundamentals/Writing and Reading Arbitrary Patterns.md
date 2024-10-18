# Writing and Reading Arbitrary Patterns

 One powerful capability of Spacely is to read and write arbitrary patterns. This allows the user to create arbitrary software-defined interfaces in Python, which can still work at the high speeds that are achievable with an FPGA.
 
 In order for this to work, there are two broad steps which are handled by two components of Spacely:
 
 1. You must convert your desired pattern into a machine-friendly vector format called a GlueWave. This is done by the **GlueConverter**.
 2. You must send your GlueWave to the hardware, and retrieve the results. This is done by the **PatternRunner.**
 
# Diagram
 
 <p align="center">
<img src="https://github.com/SpacelyProject/spacely-docs/blob/main/figures/gc-pr-diagram.PNG" width="600">
</p>
 
# Example Code

 ```python
 # Create an empty dictionary
 waves = {}
        
 # Add some signals as keys to the dictionary.
    waves["S_CLK"] = []
    waves["S_DIN"] = []

 # Programmatically define the waveform for each signal.
 sc_data = [1,0,1,0,1,1,0,0,1,1]
 for i in sc_data:
    waves["S_DIN"] = waves["S_DIN"] + [i]*2
    waves["S_CLK"] = waves["S_CLK"] + [0,1]
	
 
 # Convert your dictionary to a glue wave using the global GlueConverter instance.
 glue_wave = sg.gc.dict2Glue(waves)
 
 # Use the global PatternRunner instance to run your pattern on the hardware.
 sg.pr.run_pattern(glue_wave)
 
 ```
 
# The Vector APG Model

  <p align="center">
<img src="https://github.com/SpacelyProject/spacely-docs/blob/main/figures/vector-apg.PNG" width="600">
</p>

For both Spacely-Caribou and NI-PXI, the underlying hardware that supports arbitrary pattern generation can be conceptualized as a shift register. The collective state of all of the digital output pins of the Arbitrary Pattern Generator (APG) during a single clock cycle can be represented by a single integer (which we can call a vector). If the 0th bit of the vector is zero, output pin 0 is low; if the 0th bit of the vector is one, output pin 0 is high. At each clock cycle, the APG retrieves a new vector from its internal shift register, updating the state of all output pins. 

A Glue Wave is simply a list of integers/vectors to be placed into the APG's memory, as shown in the diagram above. 

In the case of reading signals with an APG, the operation is carried out in reverse: at each clock cycle, the actual value of each input is sampled. These sampled bits become a vector which is stored in the memory of the APG, and ultimately retrieved by software and written to a Glue Wave file. 
 
## Defining HW and I/O position: The IOSPEC file
 
When you define a signal in software suck as "S_CLK", Spacely needs two pieces of information in order to determine how to pass this signal to hardware:

1. What piece of Arbitrary Pattern Generator hardware are you referring to? 
2. Within that piece of hardware, which pin number corresponds to "S_CLK"?

The IOSPEC file answers both of these questions.

### Signal specification in an IOSPEC file
Each line within the IOSPEC file describes to a specific signal, formatted as:
``` 
{name}, {I/O}, {position}, {optional default value}
```
Where:
{name} (string) represents the signal's identifier in software (i.e. "S_CLK").
{I/O} (one char, I or O) indicates whether it is an input to or an output from the ASIC.
{position} (integer) refers to the pin number of this signal.
{optional default value} (integer, 1 or 0) is used when no signal change occurs.

### Hardware definitions for Caribou and NI-PXI

Signals are grouped by their associated hardware resource using the following construct:
```
{FPGA}/{I/O}/{INTERFACE} BEGIN 
…
END
``` 

The meanings of these fields depend on which hardware platform you are using:
**For Spacely-Caribou**
- The {FPGA} field will always be "Caribou"
- The {I/O} field will be the name of the APG firmware block that you wish to control. This should be identical to the prefix of the registers associated with that APG in your memory map. For example, if the run register is "apg1_run", this field will be "apg1".
- The {INTERFACE} field will be either "read" or "write". All Spacely-Caribou APGs have full-duplex read & write interfaces.
**For NI-PXI**
- The {FPGA} field is the name of the NI-PXI slot in which your FPGA is installed, for example "PXISlot4"
- The {I/O} field is the part number of the NI I/O card attached to the FPGA, for example "NI6583"
- The {INTERFACE} field is a string which is specific to the I/O card, and identifies one of the interfaces on that card. For example, on the NI6583 card, one interface is "se_io" referencing the single-ended I/Os available on that card.

### Example IOSPEC file
 An example Spacely-Caribou IOSPEC file:
 ```
 HARDWARE Caribou/apg/write BEGIN

CLK,I,0
SDI,I,1
LE,I,2

END

HARDWARE Caribou/apg/read BEGIN

SDO,O,0

END
```
 An example NI-PXI IOSPEC file:
 ```
 HARDWARE PXISlot4/NI6583/se_io BEGIN

//mclk and data_in are ASIC inputs on NI pins 0 and 1 
mclk,I,0
data_in,I,1

//data_out is an ASIC output on NI pin 2.
data_out,O,2

END
```
 
 
 # Glue Converter Functionality
 The Glue Converter is responsible for reading, writing, interpreting or manipulating Glue Waves. It is a purely software class with no direct interface to hardware. 
 
 Some Glue Converter capabilities include:
 - Generating Glue Waves from Python dictionaries, ASCII files, or VCD files.
 - Plotting Glue Waves graphically using matplotlib.
 - Comparing Glue Waves to each other
 - Extracting a single signal from a Glue Wave
 
 At startup, if a DEFAULT_IOSPEC is defined in your {MyASIC}_Config.py file, a global Glue Converter will automatically be initialized, which can be called as **sg.gc** (Spacely Global Glue Converter).
 
 Some Glue Converter functions can be accessed dynamically by typing **gcshell** in the main Spacely terminal. 
 
 For more information, see: [The Glue Converter](</special-topics/The Glue Converter.md>)
 
 # Pattern Runner Functionality
 The Pattern Runner is responsible for ingesting Glue Waves which describe ASIC inputs and sending them to hardware. It is also responsible for capturing ASIC outputs and transcribing them to a Glue Wave file. The Pattern Runner deals exclusively with Glue Waves, so it is typically necessary to use a Glue Converter to prepare its inputs. 
 
 There are two separate Pattern Runners, NIPatternRunner for NI-PXI and CaribouPatternRunner for Spacely-Caribou. If you define both a DEFAULT_IOSPEC and DEFAULT_FPGA_BITFILE_MAP in your {MyASIC}_Config.py file, a global NIPatternRunner will automatically be initialized. If you instantiate a Caribou instrument, a global CaribouPatternRunner will automatically be initialized. In either case, you can call the PatternRunner as **sg.pr** (Spacely Global Pattern Runner). 
 
 Source code for the Pattern Runner is found here:
 https://github.com/SpacelyProject/spacely/blob/main/PySpacely/src/pattern_runner.py
 