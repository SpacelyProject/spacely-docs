# Getting Started with Spacely 

Welcome!

Spacely is a framework for testing Application-Specific Integrated Circuits. This document is intended to help you get ready to test YOUR design with Spacely. To do that, we'll go through six steps:

1. Gathering Information about Your Design
2. Defining your Test Objectives
3. Defining your Test Interface 
4. Creating Test Hardware
5. Running Spacely for the First Time
6. Writing Spacely Test Code for your ASIC


## 1. Gathering Information about Your Design

In order for you to test your design, you need to determine some information about your design and its interfaces. 

To start testing with Spacely, you **must** have:
- A list of all pins on your ASIC
- For all analog/bias pins, the anticipated voltage and current source/sink.
- For all digital pins, the anticipated frequency of operation.
- For all digital control/readout interfaces, a detailed description of the protocol used to communicate with the interface, including a waveform timing diagram.

In addition, you **must** decide:
- Will the ASIC be tested at cryogenic temperature, and if so, what temperature and what specific cryostat will be used?
- Will the ASIC be tested under radiation, and if so, what radiation source will be used?
- Does the ASIC require connection to any type of sensor, radiation detector, or experiment to test it?

You are also encouraged to have: 
- A block diagram of all major sub-blocks in your design.
- A list of specifications and simulation results showing expected performance.
- Detailed documentation of the functional behavior of the ASIC.


## 2. Defining Your Test Objectives

A test objective is a specific measurement you want to make which will typically either confirm the ASIC's functionality, or give a result for the ASIC's figure of merit. 

Your test objectives will ultimately become your Spacely Routines.

Before writing your Spacely test, you must define test objectives, and each objective should have the following three parts:

**Input:**
- What type of signal / instrument will be required to stimulate your ASIC (pulse generator, FPGA, etc...)
- What are the specifications for the frequency, edge rate, amplitude, and noise of this signal if any?

**Output:**
- What output signal is the ASIC expected to produce as a result of this input?
- What are the specifications for an instrument to be able to capture this signal, taking into account parasitics and environmental noise?

**Measurement Procedure:**
- If this output signal is captured, what procedure will you use to generate a useful figure of merit from the raw data?
- What is the significance of the calculated result?

## 3. Defining Your Test Setup

With your design and objectives in place, you can now specify your test setup, which is a complete description of all instruments you will use and how they connect to the device under test. The recommended format for the Test Interface is an enumerated block diagram, in which your ASIC and all instruments are blocks, and every unique connection between two blocks is given a reference number, and described. 

For each interface, consider:
- Voltage range
- Speed / bandwidth
- Protocol
- Number of wires / parallel connections
- Any special hardware needed to implement the interface, such as a coupler or a balun
- Whether the interface needs to cross a temperature / vacuum gradient

All instruments needed to support the operation of your ASIC should be considered, including:
- Voltage / current biases
- Power supplies
- Input sources / Output acquisition hardware based on specs from the previous section.

Another key consideration at this point is **what instruments are supported by Spacely.** These fall into three categories:
1. **The Caribou system** is an inexpensive, all-in-one test system that include an FPGA, biases, and analog resources. To see what resources are available from Caribou, see [Caribou Specifications](</spacely-caribou/reference/CaR Board Specifications.md>)
2. **The NI-PXI system** is a modular test card system. Spacely can support NI SMUs and PSUs, as well as certain FPGA I/O cards (see [this page](</special-topics/NI-PXI Glue Firmware.md>)
3. Certain **general testbench instruments** such as power supplies, oscilloscopes, and AWGs are supported for control via GPIB or IP interfaces. There is not currently an exhaustive list of what instruments are supported -- see the **py-libs-common** repo. Almost all modern instruments come with a simple remote control interface, so if you cannot control your instrument with the existing py-libs-common code, you are encouraged to extend it.

Limited test resources have a habit of killing designers' dreams, so it is very normal to iterate between steps 1 through 3, refining your objectives to fit what can be practically tested with available hardware. :)

## 4. Creating Test Hardware

PCBs, etc. --> Section TBA

## 5. Running Spacely for the First Time

It is now time to try [Installing Spacely](</fundamentals/Installing Spacely.md>) on your machine. 

After you've done so, run through the following to see if the installation was successful:

1. Run the appropriate Spacely startup script based on your operating system (*Spacely.ps1* for Windows or *Spacely.sh* for Linux). You should see the following message:

```
No Master_Config.txt found. Creating it with default settings!
ERROR: You need to specify the name of the ASIC you wish to target in Master_Config.txt
```

2. Open the newly-created file *Master_Config.txt* and change the line *TARGET = "???"* to *TARGET = "ExampleASIC"
3. Make a new folder under *spacely/PySpacely* called *spacely-asic-config*
4. Copy the folder *ExampleASIC* from this documentation repository inside the new *spacely/PySpacely/spacely-asic-config* directory.
5. Run the Spacely startup script again. You should now see the Spacely startup banner appear, along with some other debug text, as shown below. Note that the targeted ASIC is "ExampleASIC". 

```
 * * * TARGETING "ExampleASIC" ASIC * * *
ExampleASIC has the following modules:
  - spacely-asic-config.ExampleASIC.ExampleASIC_Config
  - spacely-asic-config.ExampleASIC.ExampleASIC_Routines
 +*+*+*+*+*+*+*+*+*+*+*+*+*+*+*+*+*+
+       Welcome to Spacely!       +
+ Let's do some science together. +
+*+*+*+*+*+*+*+*+*+*+*+*+*+*+*+*+*+
```

6. For now, type **n** and press enter to skip instrument initialization. 
7. A prompt (**>**) will appear. At the prompt, type **~r0** and press enter. 
8. The message below should be displayed:

```
 `*%*` CONGRATS! `*%*` (Spacely is installed correctly.) 
```

If you see this message, Spacely is installed correctly and you may proceed. If not, return to [Installing Spacely](</fundamentals/Installing Spacely.md>) to correct any errors. 

## 6. Writing Spacely Test Code for your ASIC

With Spacely installed, you are now ready to start writing the code that will control your ASIC! the first step is to create your **project-specific configuration files.** These files will contain all of the configuration information, test routines, and helper functions that are necessary for Spacely to interact with your particular ASIC. 

The format and contents of these files are described in later sections. You can start by creating empty files, or copy the equivalent files from  *ExampleASIC* as a template. 

Spacely requires a special naming convention in order to locate and load your files. If the name of your project is "MyASIC", then you must create the subfolder /spacely-asic-config/MyASIC. Inside that folder you will create the following files:

1. MyASIC/MyASIC_Config.py -- This file will hold information about the configuration of the Test Setup you defined above, such as static settings for voltage and current biases. 
2. MyASIC/MyASIC_Routines.py -- This file will hold a collection of Python functions (your Spacely Test Routines), each of which is based on one of the Test Objectives you defined above.
3. MyASIC/MyASIC_Subroutines.py -- (Optional) If you need helper functions to simplify the implementation of test routines in MyASIC_Routines.py, you can place them here.
4. MyASIC/MyASIC_iospec.txt -- Depending on the FPGA resources you are using, you may need to write a simple txt file telling Spacely which ASIC pins are connected to which Test Setup FPGA pins.

Detailed descriptions on how to write the contents of these files are available in [Spacely Fundamentals](</fundamentals/README.md>).

Additionally, for Fermilab users, many more examples are available at: https://github.com/Fermilab-Microelectronics/spacely-asic-config

If you are using the Caribou system for your design, you will also need Caribou firmware. You can find a tutorial on getting started with Caribou [here](</spacely-caribou/README.md>)

**Note:** Once you've created these files, be sure to go back to *Master_Config.txt* and change the TARGET from "ExampleASIC" to "MyASIC". 

Once you've written the appropriate config and test routine files, testing your ASIC is as simple as launching Spacely and running the appropriate function.

Best of luck!

Refer any questions to spacelydevelopers [at] fnal [dot] gov