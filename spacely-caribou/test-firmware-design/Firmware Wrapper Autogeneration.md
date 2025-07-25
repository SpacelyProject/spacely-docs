# Autogeneration Tools for Spacely-Caribou Firmware

The Spacely-Caribou architecture requires firmware blocks to be connected to a shared AXI bus which is used to read/write data to registers in those blocks. Autogeneration tools are provided to generate this AXI interface for any new firmware block. Following this process significantly reduces the effort involved in developing new firmware blocks, and eliminates many common errors. The full process is summarized by the graphic below. 

<p align="center">
<img src="https://github.com/SpacelyProject/spacely-docs/blob/main/figures/spacely-caribou/FirmwareAutogenerationDiagram.PNG" width="700">
</p>


## fw_description Format

When an engineer writes a new firmware block (myModule.sv), they must write an accompanying myModule.fw_description file. This file is an extremely simple, machine-readable description of the I/O ports, AXI registers, and parameters of the block. 

Each parameter is described by a single line with the format:

```
PARAMETER {name} {default_value}
```

Each port (a signal that will be exposed at the top level of the block to connect to other firmware blocks or I/O pins) is described by a single line with the format:

```
PORT {name} {width} {direction}
```

Where *direction* is an integer (0=input, 1=output)

Each AXI-addressable register is described by a single line with the format:

```
REGISTER {name} {width} {type}
```

Where *type* is an integer (0=read/write, 1=read-only, 2=trigger/pulse). A trigger/pulse register delivers a pulse which is a single AXI clock cycle wide to the firmware block whenever any value is written to it. 


For an exemplary fw_description file, see the [Arbitrary Pattern Generator block](<https://github.com/SpacelyProject/spacely-caribou-common-blocks/blob/main/Arbitrary_Wave_Generator/APG.fw_description>)

## Autogenerating Firmware Wrappers from fw_description

To generate firmware wrappers from a fw_description file, run Spacely (it does not matter which ASIC Spacely is targeting) and type **gen_fw()** in the main Spacely shell. You will be prompted to give a name to your block ("myModule") and to select your myModule.fw_description file. Three files will be generated:

- **myModule_interface.sv** -- A SystemVerilog file that implements the AXI interface to your block. 
- **myModule_top.v** -- A Verilog wrapper to allow your block to be instantiated in a Vivado block diagram.
- **README.md** -- A partially-filled README file in the format specified by the spacely-caribou-common-blocks repository. This file will contain tables which describe all of the ports, registers, and parameters from the fw_description file, but it is the engineer's responsibility to fill in the details of these tables based on their knowledge of the design. 






