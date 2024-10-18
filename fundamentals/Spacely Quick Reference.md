# Spacely Quick Reference

## Command Line Options 

| Command line Option | Usage |
|---------------------|-------|
| ```--defaults```    | Initialize all instruments in the INSTR dictionary without asking for explicit permission. |
| ```-r <routine #>```| Run the routine with the given number immediately after starting Spacely. |



## Spacely Shell Commands 

Type these commands in the top-level Spacely Shell prompt (**>**).

| Spacely Shell Command | Usage |
|-----------------------|-------|
| ```help```            | Lists Spacely Shell Commands and command line options |
| ```lr```              | List all routines with their numbers and docstrings. |
| ```~r<routine #> ```  | Run the routine with the given number. | 
| ```ni_mon -a ```      | Display a live dashboard with all configured current and voltage rails. |


## Spacely Python Idioms

### Interacting with Glue Waves and Digital I/Os (Arbitrary Pattern Generation)

In most cases, all digital I/O can be carried out using the global GlueConverter and PatternRunner instances. 

Some particularly common functions are:

```python
sg.gc.read_glue(<filename>) -> GlueWave
sg.gc.write_glue(<GlueWave>, <filename>)
sg.gc.dict2Glue(<dictionary>) -> GlueWave
sg.pr.run_pattern(<GlueWave>)
```

For more information, refer to [Writing and Reading Arbitrary Patterns](/fundamentals/Writing and Reading Arbitrary Patterns.md>)

### Interacting with Voltage/Current Biases

Voltage and current biases can be referenced from using V_PORT\[\] and I_PORT\[\]: 

```python
V_PORT["VDD"].set_voltage(<voltage(volts)>)
V_PORT["VDD"].get_voltage()
I_PORT["Ibias"].set_current(<current(amps)>)
I_PORT["Ibias"].get_current()
```

The names of the biases are those defined in the [Config file.](</fundamentals/Writing an ASIC Config File.md>)

### Instrument Methods

Some instruments, such as AWGs, Oscilloscopes, or the Caribou system, have methods which can be called as:

```python
sg.INSTR["instr_name"].method(<args>)
```
Where "instr_name" is the name assigned to that instrument in the INSTR dictionary in the [Config file.](</fundamentals/Writing an ASIC Config File.md>)