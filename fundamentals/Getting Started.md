# Getting Started with Spacely 

Once you've installed Spacely, the first step is to create your **project-specific configuration files.** These files will contain all of the configuration information, test routines, and helper functions that are necessary for Spacely to interact with your particular ASIC. 

The format and contents of these files are described in later sections. You can write these files from scratch, or copy the equivalent files from the folder spacely-docs/ExampleASIC as a template. 

Spacely requires a special naming convention in order to locate and load your files. If the name of your project is "MyASIC", then you must create the subfolder /spacely-asic-config/MyASIC. Inside that folder you will create the following files:

1. MyASIC/MyASIC_Config.py -- Holds ASIC-specific settings like voltage levels, current limits, etcetera.
2. MyASIC/MyASIC_Routines.py -- Script containing the test routines.
3. MyASIC/MyASIC_Subroutines.py -- (Optional) Helper functions to be used by MyASIC_Routines.py
4. MyASIC/MyASICiospec.txt -- Defines the I/O specification (digital pinout) of your ASIC test stand. 

After creating these files, you must open the file Master_Config.py in the spacey/PySpacely, and change the global variable TARGET to the string "MyASIC" to instruct Spacely to target your project. 

# Next Steps 

Learn about each of these file types and how you can use them to start testing your ASIC:

**MyASIC_Config:**

**MyASIC_Routines/Subroutines:**

**MyASIC_iospec:** Learn more about how IOSPEC files are used to define the digital interface to your ASIC and read or write Arbitrary Patterns: (Link TBA)