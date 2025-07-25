# Working with Spacely-Caribou 

When running a Spacely-Caribou test stand, Spacely interfaces with the Caribou hardware (ZCU102, Peary server, etc.) through a Caribou() object. The Caribou() object defines some useful methods and interfaces which may be used in routines. 


## Initializing Caribou

Caribou is considered to be an instrument, which must be initialized in the INSTR dictionary of the [Config file](/fundamentals/Writing an ASIC Config File.md) like any other. Its required metadata fields are type, host, port, and device. Each of these fields has a "conventional" value which is given in the code snippet below. If you are using a standard Spacely-Caribou setup with only one Caribou system, you should stick exactly to this initialization:

```python
INSTR = {"car" : {"type": "Caribou",
                  "host":"192.168.1.24",
                  "port":12345,
                  "device":"SpacelyCaribouBasic"}}
```

**Notes:**
1. For debug purposes, it is possible to emulate the Caribou system in software. To do this, replace the host field with the string "EMULATE". 
2. On Linux systems, connecting to a Caribou device will cause Spacely to acquire a global lock on that device. This is to prevent confusion from multiple remote users simultaneously accessing a Caribou device. (This protection is not yet implemented on Windows.)

## Reading and Writing AXI Registers

Individual AXI registers can be accessed with Caribou() methods: 

```python
sg.INSTR["car"].set_memory(<reg_name>, <val>)
sg.INSTR["car"].get_memory(<reg_name>)
```

Register names should match what is supplied in the Peary memory map. 

Registers may also be accessed interactively using a Caribou() AXI shell. In order to do this, you must first supply a map of AXI registers to the Caribou() object. This map is a Python dictionary where keys are the names of firmware blocks, and values are lists of registers which fall under that firmware block. The mapping of registers to blocks is purely for organization and convenience, and doesn't affect how the registers are accessed. 

**Example Code:**

```python

REG_MAP = {"apg": ["apg_run", "apg_write_channel", "apg_read_channel", "apg_write_defaults",
				   "apg_sample_count","apg_n_samples","apg_write_buffer_len",
				   "apg_next_read_sample","apg_wave_ptr","apg_status", "apg_control",
				   "apg_dbg_error","apg_clear"],
           "logic_clk_div": ["divider_cycles","divider_rstn"]}

sg.INSTR["car"].axi_registers = REG_MAP

//Invoke the interactive shell
sg.INSTR["car"].axi_shell()

``` 

## Controlling Voltage and Current Supplies

Caribou voltage and current supplies should be set up and controlled using the V_PORT/I_PORT method described in [Writing an ASIC Config File](/fundamentals/Writing an ASIC Config File.md). The "channel" name referred to in that section is the Caribou resource name, for example "PWR_OUT_1" or "BIAS1".

**NOTE:** As of 10/17/2024, it is confirmed that Spacely-Caribou control works for "PWR_OUT" resources, but other resources may or may not work.


## Setting CMOS Level

The following methods can be used to set the input/output CMOS voltage level for all channels:

```python
sg.INSTR["car"].set_output_cmos_level(<voltage>)
sg.INSTR["car"].set_input_cmos_level(<voltage>)
```

