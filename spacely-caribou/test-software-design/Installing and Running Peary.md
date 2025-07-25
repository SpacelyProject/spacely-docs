# Installing Peary


1. Clone the peary repo from the **aq_dev** branch of: https://gitlab.cern.ch/adquinn/peary/-/tree/aq_dev?ref_type=heads 
2. SCP the Peary repo to your ZCU102
3. You need **cmake** to build Peary, which you can download from the internet. SCP the cmake installer script (i.e. ```cmake-3.29.0-rc4-linux-aarch64.sh``` onto the ZCU102
4. Install cmake on the ZCU102 by running the installer script. 
5. Make a directory inside peary called "build", i.e. ```/peary/build```, and cd into that build folder.
6. Run: ```cmake –DBUILD_SpacelyCaribouBasic=ON –DINTERFACE_IIO=OFF –DINTERFACE_MEDIA=OFF ..```

**NOTE:** If you are running without a CaR board attached to the ZCU102, also include the flag ```–DINTERFACE_I2C=OFF```, otherwise it will fail trying to get the CaR board serial number from EEPROM.


# Building Peary

**Important NOTE:** You need to rebuild Peary every time you make a modification to the Peary device files.

Still inside the build folder, run the following commands:

1. ```make -j4```
2. ```sudo make install```

**NOTE:** If you see "your build may be incomplete" or "clock-skew detected" this is caused by the fact that the system clock resets every time that the board resets, causing cmake to believe that the modification dates for certain files are actually in the future. To fix this, you can advance the system clock, e.g. by ```sudo date --set='+1day’```

Alternative: Reset the last-modified dates of appropriate files by touching them: ```sudo find . -type f -exec touch {} +```


# Running Peary

To run peary as a CLI (useful for debugging if you haven't set up Spacely yet): ```sudo ./bin/pearycli```

Try the following:
- ```# add_device SpacelyCaribouBasic```
- ```# listMemories 0```
This should return the list of the FPGA regs defined in your memory map.

To run peary as a daemon (which Spacely can communicate with): ```sudo ./bin/pearyd```

To print additional debug information (at the cost of slowing down Peary substantially), include the flag: ```-v TRACE```