# ASIC Test Routines and Subroutines

Arguably the most important file that you will write to test your ASIC with Spacely is *spacely-asic-config/MyASIC/MyASIC_Routines.py* (where "MyASIC" is replaced by the name of your device). In this file, you will write test routines that manipulate the control signals of your ASIC and monitor the results. 

Your test routines should be written as functions with names that begin with the word "ROUTINE_":

```python
def ROUTINE_{name}(): 
```
When you run Spacely, it will read your Routines file and assign a number to each routine. You will see comments appear just before the routine name which look like this:

```python
#<<Registered w/ Spacely as ROUTINE 0, call as ~r0>>
```
This means that you can call the routine from the main Spacely shell by typing **~r0**. In general, routine N may be called by typing **~rN**. 

For complex routines, you may want to call helper functions, which may be called any name, and should be written in a separate file called *spacely-asic-config/MyASIC/MyASIC_Subroutines.py*. Subroutines will not be assigned numbers. However, if needed, you can call any Routine or Subroutine from the Spacely shell by using its name. 

## Guidelines for Writing Routines 

**Routines are documentation.** In addition to actually carrying out a test, a Routine serves as precise documentation of exactly what test was carried out. Thus Routines should be written in a clear and explicit fashion, with self-explanatory variable names and comments where needed. 

**Routines should be self-sufficient.** In general, a Routine should accomplish a single, high-level task which does not depend on other routines being run or arguments being passed in. *ROUTINE_Plot_ADC_Nonlinearity()* is likely a good routine, while *ROUTINE_send_spi_command(addr, cmd)* is probably too low-level to be a routine, and should be converted to a subroutine. This is important for simplicity, reproducibility, and documentation. In general, you should only need to call one routine, A(), from the Spacely shell in order to accomplish a certain task. If you need to call A(), then B(), then C(), this creates an procedure that the user must remember or document inside of Spacely. A better practice in this situation is to make A(), B(), and C() subroutines, and then write a routine X() which calls them in order. In this case, X() will contain complete documentation of the test which is carried out. 

**Routines should use an appropriate level of abstraction.** You should consider the relative performance and ease of modification for different layers in the Spacely stack when writing optimal routines. See [Appropriate Use of Abstraction](</special-topics/philosophy/Appropriate Use of Abstraction.md>) for more information. 


## Spacely Idioms

Spacely code idioms and functions are necessary to write efficient and readable Spacely code. See [Spacely Quick Reference](</fundamentals/Spacely Quick Reference.md>)