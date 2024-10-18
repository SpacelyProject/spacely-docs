# Getting Started with Spacely 

## Running Spacely for the First Time

1. Open the file *Master_Config.py* in the folder *spacey/PySpacely*, and ensure that the global variable TARGET is set to the string "ExampleASIC".
2. Run the appropriate startup script based on your operating system (*Spacely.ps1* for Windows or *Spacely.sh* for Linux). You should see the Spacely startup banner appear, as shown below. Note that the targeted ASIC is "ExampleASIC". 
<p align="center">
<img src="https://github.com/SpacelyProject/spacely-docs/blob/main/figures/ExampleASIC_Startup.PNG" width="700">
</p>

3. For now, type **n** and press enter to skip instrument initialization. 
4. A prompt (**>**) will appear. At the prompt, type **~r0** and press enter. 
5. The message below should be displayed:

```
 `*%*` CONGRATS! `*%*` (Spacely is installed correctly.) 
```

If you see this message, Spacely is installed correctly and you may proceed. If not, return to [Installing Spacely](</fundamentals/Installing Spacely.md>) to correct any errors. 

## Project-Specific Configuration Files

With Spacely installed, you are now ready to start writing the code that will control your ASIC! the first step is to create your **project-specific configuration files.** These files will contain all of the configuration information, test routines, and helper functions that are necessary for Spacely to interact with your particular ASIC. 

The format and contents of these files are described in later sections. You can write these files from scratch, or copy the equivalent files from the folder *spacely-docs/ExampleASIC* as a template. 

Spacely requires a special naming convention in order to locate and load your files. If the name of your project is "MyASIC", then you must create the subfolder /spacely-asic-config/MyASIC. Inside that folder you will create the following files:

1. MyASIC/MyASIC_Config.py -- Holds ASIC-specific settings like voltage levels, current limits, etcetera.
2. MyASIC/MyASIC_Routines.py -- Script containing the test routines.
3. MyASIC/MyASIC_Subroutines.py -- (Optional) Helper functions to be used by MyASIC_Routines.py
4. MyASIC/MyASICiospec.txt -- Defines the I/O specification (digital pinout) of your ASIC test stand. 

Once you've created these files, be sure to go back to *Master_Config.py* and change the TARGET from "ExampleASIC" to "MyASIC". 


**Next Steps:** Return to [Spacely Fundamentals](</fundamentals/README.md>) to learn more about what you should write inside these files.
