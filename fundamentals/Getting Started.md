# Getting Started with Spacely 

## Running Spacely for the First Time

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

## Project-Specific Configuration Files

With Spacely installed, you are now ready to start writing the code that will control your ASIC! the first step is to create your **project-specific configuration files.** These files will contain all of the configuration information, test routines, and helper functions that are necessary for Spacely to interact with your particular ASIC. 

The format and contents of these files are described in later sections. You can write these files from scratch, or copy the equivalent files from  *ExampleASIC* as a template. 

Spacely requires a special naming convention in order to locate and load your files. If the name of your project is "MyASIC", then you must create the subfolder /spacely-asic-config/MyASIC. Inside that folder you will create the following files:

1. MyASIC/MyASIC_Config.py -- Holds ASIC-specific settings like voltage levels, current limits, etcetera.
2. MyASIC/MyASIC_Routines.py -- Script containing the test routines.
3. MyASIC/MyASIC_Subroutines.py -- (Optional) Helper functions to be used by MyASIC_Routines.py
4. MyASIC/MyASIC_iospec.txt -- Defines the I/O specification (digital pinout) of your ASIC test stand. 

Once you've created these files, be sure to go back to *Master_Config.txt* and change the TARGET from "ExampleASIC" to "MyASIC". 


**Next Steps:** Return to [Spacely Fundamentals](</fundamentals/README.md>) to learn more about what you should write inside these files.
