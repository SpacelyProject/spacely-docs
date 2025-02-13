# Global Variables and Scope in Spacely

Some use of global-scoped functions and variables is needed in Spacely to share information between Spacely core modules and user-defined modules. There are two broad kinds of variable which have some global scope:

1. Spacely Global Variables
2. User-defined Functions and Variables

## Spacely Global Variables

Spacely Global Variables are variables and data structures which are needed for the operation of Spacely core modules. 

Since v0.2.0, the *only* appropriate vehicle for Spacely Global Variables is the Spacely_Globals.py module. Any file which needs to use these variables should use the idiom ```import Spacely_Globals as sg```. Variables can now be accessed as ```sg.<VariableName>```.

This arrangement ensures:
1. Consistency of these Spacely Global Variables across the different modules of Spacely. These variables technically reside in the scope of the Spacely_Globals.py module, so there is no need to worry about the fact that in Python each file has its own separate global scope.
2. Non-pollution of the global namespaces of each individual file, and thus non-collision with user variables (all variables are scoped under "sg").

### Configuring Spacely Global Variables

Default values for Spacely Global variables can be defined at the top level of the repository in **Master_Config.txt** as ```VARIABLE_NAME=Value```. If this file does not exist, it will be automatically created the first time that you run Spacely. 

**In particular, one variable MUST be defined in Master_Config.txt:** The variable ```TARGET``` defines which ASIC configuration is loaded from **spacely-asic-config**.

Values of these variables can be re-defined per-ASIC by including the lines ```import Spacely_Globals as sg``` and ```sg.VARIABLE_NAME = Value``` in your **MyASIC_Config.py** file. 

## User-defined Functions and Variables

Global variables and functions which are defined in the user-contributed files (MyASIC_Routines, MyASIC_Subroutines, and MyASIC_Config) are imported into the global scope by Master_Config.py. This ensures that they are usable between user-contributed files and in the Spacely command line. For any file that you want to be able to access these variables, make sure to use the idiom ```from Master_Config import *```

One notable instance when these variables are *expected* to be defined by the Spacely core modules is the INSTR / SEQUENCE / CHAN variables used to set up voltage and current bias channels. 