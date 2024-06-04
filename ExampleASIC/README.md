# Example ASIC

This directory contains the three basic files (Config, Routines, and iospec) which you need to customize Spacely for your ASIC.

Notice how the name of the directory, and all three files, starts with "ExampleASIC". You can change this to the name of your ASIC, but be sure to follow the convention.


## Verifying your Spacely Install

Once you've installed Spacely as explained in the Docs, try this:

1. Copy this directory into your spacely install directory at: spacely/PySpacely/spacely-asic-config/ExampleASIC 
2. Open the file spacey/PySpacely/Master_Config.py and make sure you see the line TARGET="ExampleASIC"
3. Run Spacely, skipping instrument initialization (./Spacely.ps1 --skipall or ./Spacely.sh --skipall)
4. Run Routine 0 from the terminal by typing "~r0"
5. If you see the word "CONGRATS!" then you've successfully installed Spacely.

![Screenshot of successful install](./spacely_install_screenshot.PNG)