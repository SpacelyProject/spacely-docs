# Writing Instrument Drivers for Spacely

To create a new instrument driver for use in Spacely, following the following steps.

## In py-libs-common

1. Create a Python class for the instrument in a new file under libinstrument/fnal_libinstrument
2. If the I/O method you want to use to control the instrument is not already defined under libIO/fnal_libIO, create it as well.
3. Remember for both libinstrument and libIO to include the new classes in the list of symbols to export in ```__init__.py```

## In Spacely_Utils

1. Edit the linting dictionaries ```instr_type_required_fields``` and ```io_required_fields``` to reflect the requirements for your new instrument.
2. Create a new case in the ```initialize_INSTR``` function to set up an instrument of your new type.
3. Create a new case in the ```deinitialize_INSTR``` function to safely shut down the instrument. 

