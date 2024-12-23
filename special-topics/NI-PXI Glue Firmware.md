# NI-PXI Glue Firmware

The following configurations are supported:

| FPGA I/O Module	| FPGA Card Version |	Bitfile Clock Speed	| FIFOs	| Spacely Bitfile Name |
|-------------------|-------------------|-----------------------|----|----|
| NI6583 | NI7976 |	40 MHz	| se_io (32b) lvds (32b) |	“NI7976_NI6583_40MHz” |
| NI6583 |	NI7972 |	40 MHz |	se_io (32b) lvds (32b) |	“NI7972_NI6583_40MHz” |
| NI6581 | NI7962  | 40 MHz	 | tba | “NI7962_NI6581_40MHz” |
| NI6581 | NI7972  |  N/A   | N/A | N/A – This configuration is not compatible. |


**Choosing a FIFO**

Glue uses internal FIFOs to pass data from computer memory to the ASIC and back again, ensuring that your waveforms can run at a sufficient rate. Each FIFO corresponds to a specific group of digital I/O signals. 

**NI6583**
- The **se_io** FIFO is used to control single-ended I/Os. There are 32 controllable signals, and read/write direction can be set for each signal independently.
- The **lvds** FIFO is used to control LVDS I/Os. There are 32 controllable signals, and read/write direction can be set for each signal independently. 
