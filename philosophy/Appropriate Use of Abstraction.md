# The Appropriate Use of Abstraction 

When testing an electronic circuit or an ASIC, we are faced with the challenge of having to interact with the DUT (Device Under Test) using tools. A human cannot manually operate a communications bus faster than 1 Hz, much less 1 MHz. Nor do we care to manually apply 1,000 distinct test input voltages and monitor the output voltage for each. In each case, we create tools to do these tasks for us.

Whenever we create a tool, we create a layer of abstraction. The user will type a series of numbers into an IDE, and we assume that, upon running the program, those numbers cause specific bit flips in the hardware drivers of the I/O pins of our test instrument. In most cases, there are actually multiple layers of abstraction in between the human and the hardware, forming a layer cake something like the familiar [TCP/IP protocol stack](https://www.geeksforgeeks.org/tcp-ip-model/).

However, unlike a consumer using TCP/IP, a test engineer typically must develop at least some of the layers in their tool stack, debug these layers and iterate on them as necessary. The time cost of developing these tools is NOT trivial -- it often dominates the total time taken to test the DUT. Thus we are presented with an optimization problem: How many tools shall we build? What layer of abstraction shall we operate on, and is it the same for every task? 

Consider the simple task of sending a hexadecimal word (0xDEADBEEF) over a SPI link to a DUT. The figure below presents four approaches to the problem, with different tool stacks (a-d). 

<p align="center">
<img src="https://github.com/SpacelyProject/spacely-docs/blob/main/figures/AbstractionApproaches.PNG" width="600">
</p>

Approach (a) is clearly nonsensical. Who would build an entire hardware block just to send one specific word? (b) is more reasonable, but still likely requires a re-compilation of a C++ codebase in order to change the word being tested. (c) removes this requirement by shifting the application-specific information to Python, a high-level interpreted language. (d) removes application-specific information from the codebase entirely; the user may input any test value they choose. 

We may make some general observations:

- Moving up the layers of abstraction, **iteration time** sharply decreases. (It may take several minutes to rebuild a C++ codebase, but only seconds to type in a new value into a user-interface.)

- When we create more layers of abstraction, we create more **reusable tools**. The hardware and C++ blocks in (c-d) can easily be reused for testing another similar SPI enabled ASIC. 

- Moving up the layers of abstraction, **execution time** increases. (It takes more time for a human to type 0xDEADBEEF than for a CPU to read the same word from program memory.)


**A Key Observation: Up to the Python level, iteration time almost always dominates over execution time**. This means that it is almost always advantageous to keep application-specific information at the Python level of abstraction, rather than hardcoding it into lower levels of abstraction. This is the primary rationale for the existence of Spacely. 

The justification of this observation is simple: iteration is an engineering process, in which the slowest part is the human engineer taking time to understand the problem and propose a solution, then design tools/code to implement that solution. Execution, irrespective of programming language, takes place in fractions of a second. 

When the application-specific information is abstracted into the user domain using a user interface (i.e. as in (d)), this trade-off becomes less clear. Now that the human is part of both the iteration and the execution process, it is no longer certain that iteration time will dominate, and the correct trade-off depends on the details of the situation. For example:

- If you are debugging a SPI interface to prove basic functionality for the first time, approach (d) may make more sense: You would expect to iterate often, but you only need to execute once or twice to prove success or failture.

- If you are sweeping through all possible commands on a SPI interface, then a version of approach (c) where the argument to SPI.write() is assigned using a for-loop is appropriate, and much more time-efficient than having a user type all possible values. 

Another drawback of (d) is that the specific commands the user entered through the user interface are not recorded. Whereas, if a for loop is written in Python code, this for loop is a self-documenting record of the test that was performed. 


## Caching 

It should be noted that in many cases, while the execution time of Python is not a problem when compared to iteration time, it is too slow to directly interact with the DUT. For example, if the DUT requires data over a SPI bus at 10 MHz, sending each bit with an individual Python command will be far too slow. 

In almost all cases, this can be resolved by some form of caching: A set of bits is written into FPGA memory from Spacely, taking a leisurely amount of time, and once all bits are collected in the FPGA "cache", a transaction is triggered and those bits are transmitted at 10 Mb/s until they are exhausted. See the spacely-caribou-common-blocks [Arbitrary_Pattern_Generator](https://github.com/SpacelyProject/spacely-caribou-common-blocks/tree/main/Arbitrary_Wave_Generator) and [simple_serial](https://github.com/SpacelyProject/spacely-caribou-common-blocks/tree/main/simple_serial) for example implementations. 


## Levels of Abstraction within Spacely 

Naturally, "Python" is not exactly one level of abstraction. The Spacely codebase actually contains approximately four levels of abstraction:

1. **MyASIC_Routines.py** -- (User Created) Contains high-level test routines; this file should be understood as the written instructions from the test engineer to the machine, which are highly application-specific and non-reusable. 
2. **MyASIC_Subroutines.py** -- (User Created) Contains helper functions which perform defined, reusable tasks for your specific ASIC.
3. **Spacely Core Code** -- Spacely.py and Spacely_Utils.py utility functions that should work for many ASICs.
4. **py-libs-common** -- Library functions, which are sometimes specific to test instruments, but should not be ASIC-specific. 


## Arbitrary Pattern Generation   

In the example up until now, we have considered that the spi data (0xDEADBEEF) is the only piece of the tool stack which may need to be changed. However, in practice, there is at least one other component which may be application-specific: the SPI protocol itself! Some ASICs may implement variations on the SPI protocol, or similar custom serial protocols. In the models above, this would potentially require a revision of all levels of the stack. 

Fortunately, Spacely provides a way around this through Arbitrary Pattern Generation. The idea is that SPI, serial, I2C, etc are really just application-specific instances of a much more general arbitrary protocol, in which *N* distinct digital outputs are modulated with a defined phase relationship, and *N* distinct inputs are recorded synchronously. Using this abstraction, a SPI write can be represented as a list of *K* samples, where each sample describes the state of the *pico* and *cs* signals at a single SPI clock rising edge. 

Our stack is now:
<p align="center">
<img src="https://github.com/SpacelyProject/spacely-docs/blob/main/figures/AbstractionApproaches_APG.PNG" width="400">
</p>

Arbitrary Pattern Generation is provided in the Spacely-Caribou flow by the [Arbitrary_Pattern_Generator](https://github.com/SpacelyProject/spacely-caribou-common-blocks/tree/main/Arbitrary_Wave_Generator) firmware block, and in the NI PXI flow by the [PatternRunner](https://github.com/SpacelyProject/spacely/blob/main/PySpacely/src/pattern_runner.py) Spacely class. 

The use of Arbitrary Pattern Generation does introduce additional overhead due to the need to compile a pattern, but it is often justified by the saved time of not developing (and debugging) a protocol-specific firmware block.


## Key Guidelines for Spacely-Caribou Development

When developing for Spacely-Caribou, use of the proper abstraction layers is key to successful development. In particular:

- Firmware blocks should perform a single well-defined, ASIC-independent function. 

- Peary device files and Spacely libraries should be ASIC-independent. 

- ASIC-specific information should be kept, to the greatest extent possible, within the spacely-asic-config files (MyASIC_Routines, Subroutines, Config, and iospec)


_NOTE:_ There are some exceptions to the best practices described here with respect to Peary device files. Specifically, memory maps and Si5345 configurations are application-specific despite residing on the C++ level of abstraction. Fortunately, both of these may be autogenerated, mitigating the impact on iteration time.



