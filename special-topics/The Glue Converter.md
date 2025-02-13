# The Glue Converter

## Summary

 The Glue Converter is responsible for reading, writing, interpreting or manipulating Glue Waves. It is a purely software class with no direct interface to hardware. 
 
 Some Glue Converter capabilities include:
 - Generating Glue Waves from Python dictionaries, ASCII files, or VCD files.
 - Plotting Glue Waves graphically using matplotlib.
 - Comparing Glue Waves to each other
 - Extracting a single signal from a Glue Wave
 
 
## Invoking Glue Converter functions. 

 At startup, if a DEFAULT_IOSPEC is defined in your {MyASIC}_Config.py file, a global Glue Converter will automatically be initialized, which can be called as **sg.gc** (Spacely Global Glue Converter).
 
 In Routines, methods can thus be invoked as:
 ```
 sg.gc.method(<args>)
 ```
 
 Some Glue Converter functions can also be accessed dynamically from a shell which can be activated by typing **gcshell** in the main Spacely terminal. 
 
 Common Glue Converter functions are given below. Where applicable, the equivalent syntax is given to access the function either from Python or from **gcshell**. 
 
## Creating Glue Waves 
 
### Creating Glue Waves from Python (Dictionaries)
 
Often the simplest way to create a GlueWave is to programmatically create a Python dictionary where each key of the dictionary is a signal name matching a signal from the IOSPEC, and each value is a list of the binary values (1 or 0) taken by that signal at each timestep. Once created, this dictionary can be converted to a GlueWave by invoking:  

- gc.dict2Glue()

**Code Example:**
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
 ```
 
 
### Creating Glue Waves from ASCII

It's also possible to define GlueWaves by ASCII in a txt file. This file can then be ingested by:

- gc.ascii2Glue() or **gcshell>ascii2input** or **gcshell>ascii2golden**. 

(See "Input" vs "Golden" waves below)

**Example ASCII File:**
```
Clk_a: 0011001100110011001100110011000000000
Clk_b: 0001110001110001110001110001110000000
Data:  0000001111110000001111111111110000000
```

**Example Code:**
```
glue_wave = sg.gc.ascii2Glue("my_wave.txt")
```

### Creating Glue Waves from VCD Files
 
Value Change Dump (VCD) is a common format for exporting waves from RTL simulation. These waves can be directly used as stimulus vectors for Spacely. 

The full procedure for exporting and using a VCD wave is as follows (assuming that you use Cadence SimVision for simulation):

1. Run the top-level testbench in SimVision.
	1. Use the ‘make sim’ command to run the testbench.
	2. Select objects in the top-level testbench and send them to the target waveform window  
	3. Run simulation in the SimVision window.
2. Choose an appropriate time interval and signals to export depending on the test goals. Exporting too many signals or too long of a time interval may make the Glue Wave needlessly large, slowing down the test.
3. Export to a VCD file.
	1. Confirm selected variables.
	2. Set specified times.
	3. Database Format: Value Change Dump
4. Transfer the VCD file to the computer running Spacely.

Finally, convert the VCD to a GlueWave using: 
 
- gc.VCD2Glue() or **gcshell>vcd2input** or **gcshell>vcd2golden**.

(See "Input vs Golden waves", and "strobe_ps" sections below)
 
## Reading and Writing Glue Waves
 
Glue waves can be read and written with:

- gc.read_glue()  or **gcshell>getglue**
- gc.write_glue() or **gcshell>writeglue**


## Plotting Glue Waves

Spacely can plot Glue Waves with Matplotlib:

- gc.plot_glue() or **gcshell>plotglue**

The **gcshell** function will either plot the most recently loaded GlueWave, or if none have been loaded, it will prompt you to load a new one. 


## Manipulating Glue Waves

To extract a single signal from a GlueWave, use:

- gc.get_bitstream() or **gcshell>bits**


## More Information:

Glue Converter Source Code: https://github.com/SpacelyProject/py-libs-common/blob/main/nitoolbox/src/fnal_ni_toolbox/glue_converter.py

Glue Converter Test Suite: https://github.com/SpacelyProject/py-libs-common/blob/main/nitoolbox/src/fnal_ni_toolbox/glue_converter_test.py


### "Input" vs "Golden" Waves

For ASCII and VCD to GlueWave conversion, the Glue Converter can generate either an "input" wave which contains only ASIC input signals (as would be appropriate for a stimulus vector), or a "golden" wave which includes both ASIC input and output signals (as would be appropriate for a golden reference, which the actual measured ASIC outputs will be compared against). 

In Python, this is achieved by setting the *inputs_only* argument (defaults to True). In **gcshell**, two commands are provided, an input and golden version. 


### strobe_ps

When creating Glue Waves from VCD files, *strobe_ps* is a scaling parameter which helps conversion between simulation and test hardware.

Functional RTL simulations may be run with an arbitrarily large or small timebase, and signals may change at any time. However, Spacely's FPGA operates with a fixed clock, outputting one new sample every clock cycle. This means that somehow the VCD waveform must be converted into series of samples with fixed spacing in time. 

The *strobe_ps* parameter tells the Glue Converter what strobe period it should use to sample your VCD waveform, with each sample being converted into one sample on the FPGA. 

For example, if you set *strobe_ps=100*, then the first sample in your GlueWave will be the state of all signals at t=0 in your simulation, the second sample will be the state of all signals at t=100ps, the third sample will be at t=200ps, and so on. If your VCD waveform contains signals that oscillate faster than the Nyquist sampling frequency of (1/100ps)/2 = 5 GHz, these signals will likely be distorted or lost. 

Note that setting *strobe_ps* has no effect on the clock of your actual FPGA. If you set *strobe_ps=100*, but your FPGA clock has a frequency of 40 MHz = 25000 ps, then your wave will effectively run 250x slower in real life than in simulation. 

If you want your wave to run exactly as fast in the real world as in simulation, you should set *strobe_ps* equal to your FPGA clock period in picoseconds. However, you must be mindful of the Nyquist constraint. 

When Glue Converter runs a VCD-to-Glue conversion, it will print out the mathemtaical conversions from VCD time to real time based on your *strobe_ps* value to enable you to double-check this.