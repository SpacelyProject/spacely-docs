# Writing an ASIC Config File

Your ASIC config file provides static configuration information which describes your test setup and instruments to Spacely.

It has two main parts: the Instrument Dictionary and Voltage / Current Bias Configuration.

## Instrument Dictionary

Your ASIC configuration file should define a dictionary named *INSTR* which describes the instruments in your test setup. 

Each key in the dictionary is a keyword string which is used to reference that instrument. Each value is a dictionary which contains several fields of metadata about that instrument. 

Each instrument, at minimum, has a "type" field which describes what type of instrument it is. Other required fields are described below:

| Instrument Type | Description | Required Fields |
|-----------------|-------------|-----------------|
| NIDCPower       | National Instruments DCPower Type Card | type, slot |
| Oscilloscope    | Oscilloscope | type, io |
| AWG    | Arbitrary Wave Generator | type, io |
| Supply    | Generic (non-NI) Power Supply | type, io |
| Caribou   | Caribou System | type, host, port, device |


| Field | Description |
|-------|-------------|
| type  | Instrument type, must take one of the values from the table above. | 
| slot  | The slot occupied by a card in a NI-PXI chassis. Often but not always in the form PXI{a}Slot{b}. Use NI-MAX software to confirm. | 
| io    | The type of I/O interface to a benchtop instrument. Current supported values are "VISA" or "Prologix". If an instrument has "VISA" type I/O, it must also have a "resource" field. If an instrument has "Prologix" type I/O, it must also have a "gpibaddr" field. |
| resource | VISA resource name for connecting to an instrument. Use PyVISA to identify. | 
| ipaddr   | IP Address for connecting to an instrument w/ Prologix GPIB bridge. |
| gpibaddr | GPIB Address for connecting to an instrument w/ Prologix GPIB bridge. | 
| host     | Hostname of the ZCU102 (Spacely-Caribou). | 
| port     | Port on which the Peary server runs (Spacely-Caribou). |
| device   | Name of the peary software device, typically "SpacelyCaribouBasic" (Spacely-Caribou). | 

An example instrument dictionary for an NI-PXI Test Setup:

```python
INSTR = {"SMU_A" : {"type" : "NIDCPower", 
                    "slot" : "PXI1Slot2"},
         "SMU_B" : {"type" : "NIDCPower",
                    "slot" : "PXI1Slot3"},
         "PSU_A" : {"type" : "NIDCPower",
                    "slot" : "PXI1Slot7"},
         "PSU_B" : {"type" : "NIDCPower",
                    "slot" : "PXI1Slot8"},
         "Scope" : {"type" : "Oscilloscope",
                    "io"   : "VISA",
                    "resource" : "USB0::0x0957::0x1745::MY48080042::INSTR",
                    "alias": "scope"},
         "AWG"   : {"type" : "AWG",
                    "io"   : "VISA",
                    "resource" : "GPIB1::10::INSTR"
                    },
         "PSU_C" : {"type" : "Supply",
                    "io"   : "Prologix",
                    "ipaddr" : "192.168.1.15",
                    "gpibaddr" : 5}
        }
```

An example instrument dictionary for a Spacely-Caribou Test Setup:

```python
INSTR = {"car" : {"type": "Caribou",
                  "host":"192.168.1.24",
                  "port":12345,
                  "device":"SpacelyCaribouBasic"}}
```


## Voltage / Current Bias Configuration 

Create the following variables to define voltage / curent bias rails for your ASIC. These rails will be automatically initialized when you power on the ASIC. 

See ExampleASIC_Config.py for examples. 

Voltage Biases:

| Variable Name | Type | Acceptable Values | 
|---------------|------|-------------------|
| V_SEQUENCE    | list \[str\] | List of unique string names for voltage rails, like "VDDA", "Vref_adc", etcetera. These string names are used as keys in the following dictionaries. The order of this list is the order in which rails will be initialized. |
| V_INSTR       | dict {str:str} | Key: voltage rail name, Value: Instrument that supplies the rail (key from INSTR dictionary) |
| V_CHAN        | dict {str:int/str} | Key: voltage rail name, Value: Instrument Channel that supplies the rail (channel # or Caribou channel name) | 
| V_LEVEL       | dict {str:float} | Key: voltage rail name, Value: voltage (in Volts) to be supplied on that channel. |
| V_WARN_VOLTAGE | dict {str:\[float, float\] | Key: voltage rail name, Value: List of two values which represent the high and low voltage warning levels. If the voltage on the rail falls below the former or above the latter, Spacely will generate a warning. |
| V_CURR_LIMIT  | dict {str:float} | Key: voltage rail name, Value: Current limit for the voltage rail in amperes. |
| V_PORT        | dict {str:None}  | Key: voltage rail name, Value: None (Python will replace this None with a reference to the voltage rail object once it is instantiated. |

Current Biases:

| Variable Name | Type | Acceptable Values | 
|---------------|------|-------------------|
| I_SEQUENCE    | list \[str\] | List of unique string names for current biases, like "Ib1", "Ib2", etcetera. These string names are used as keys in the following dictionaries. The order of this list is the order in which biases will be initialized. |
| I_INSTR       | dict {str:str} | Key: current bias name, Value: Instrument that supplies the bias (key from INSTR dictionary) |
| I_CHAN        | dict {str:int/str} | Key: current bias name, Value: Instrument Channel that supplies the rail | 
| I_LEVEL       | dict {str:float} | Key: current bias name, Value: current (in Amperes) to be supplied on that channel. |
| I_WARN_VOLTAGE | dict {str:\[float, float\] | Key: current rail name, Value: List of two values which represent the high and low voltage warning levels. If the voltage on the rail falls below the former or above the latter, Spacely will generate a warning. |
| I_VOLT_LIMIT  | dict {str:float} | Key: current bias name, Value: Voltage limit for the current rail in volts. |
| I_PORT        | dict {str:None}  | Key: current bias name, Value: None (Python will replace this None with a reference to the current bias object once it is instantiated. |


## Other Configuration Values

**DEFAULT_IOSPEC** -- This variable specifies a default IOSPEC file to be loaded. Typically it is set to the path to your IOSPEC file, starting from the /PySpacely/ directory, for example: 

```
DEFAULT_IOSPEC = ".\\spacely-asic-config\\SPROCKET2\\sprocket2_iospec.txt"
```

**DEFAULT_FPGA_BITFILE_MAP** -- For NI-PXI, this variable specifies the bitfile to load onto each FPGA slot that you are using. For example:  
```
DEFAULT_FPGA_BITFILE_MAP = {"PXI1Slot4":"NI7972_NI6583_40MHz"}
```

For information on supported bitfiles, see [NI-PXI Glue Firmware](</special-topics/NI-PXI Glue Firmware.md>)

**USE_NI** -- Set this variable to true to automatically initialize an NI Pattern Runner. 