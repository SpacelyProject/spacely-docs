# CaR Board Specifications

**GENERAL NOTES:**
1. **Noise Propagation:** Significant noise is known to propagate to CaR board bias outputs from the switching regulator of the ZCU102. This noise may be worse when the HPC0 connector is used to connect to the CaR board. Work is underway to characterize the magnitude of this noise and establish mitigations. 
2. **Peary Dispatcher:** Resources must be added to the dispatcher in your Peary cpp device file in order to be used. 

## Power Supplies (PWR_OUT)

| Specification | Value |
| --- | --- |
| **# of Channels** |  8 |
| **Voltage Range [V]**| 0.8 - 3.6 |

**Notes:** 
1. **Current Measurement:** The power supplies measure current with a 0.01 ohm feedback resistor, and the monitoring component has an input offset of up to 10 uV, which may cause significant issues when measuring currents of ~ a few mA or smaller. See https://www.ti.com/product/INA226. One solution is to swap the feedback resistor for a higher value.
2. **Offset:** Power supplies may exhibit an offset on the order of 50 mV (Source: BNL testing). The monitor should report the correct voltage. 


## Voltage References (BIAS)

| Specification | Value |
| --- | --- |
| **# of Channels** |  32 |
| **Component** | DAC7678 |
| **Voltage Range [V]**| 0 - 4 |
| **Voltage Resolution [mV]**| ~1 (12 bit resolution from 4.0V reference) |
| **Output Noise**| <150 nV/sqrt(Hz) above 40 Hz (see datasheet Fig 48) |
| **Short Circuit Output Current [mA]**| 25 |

**Notes:**
1. **Load Stability:** The voltage references can only drive a small load capacitance (< 1nF). Attaching a larger capacitance may result in instability and oscillation -- see DAC7678 datasheet for details.

## Current References (CUR)

| Specification | Value |
| --- | --- |
| **# of Channels** |  8 |
| **Component** | Discrete OpAmp / MOSFET circuit |
| **Current Range [A]**| 0 - 1 mA |
| **Current Accuracy [A]**| Measured <1 uA (at BNL) |

**Notes:**
1. **STILL IN DEVELOPMENT** -- Spacely v0.2.0 does not support control of CaR board current references.
2. **Max Output Voltage:** If unloaded, current sources will pull their output to 3.3 Volts. 

## Slow ADC Channels (VOL_IN)

| Specification | Value |
| --- | --- |
| **# of Channels** |  8 |
| **Component** | ADS7828 |
| **Voltage Range [V]**| 0 - 4 |
| **Input Capacitance**| ~ 1 nF |

**Notes:**
1. The input capacitance of slow ADC channels is dominated by a 1nF filter cap on the CaR board. The input capacitance of the ADS7828 alone is 25 pF. 


## Fast ADC Channels (ADC_IN)

| Specification | Value |
| --- | --- |
| **# of Channels** |  16 |
| **Voltage Range [V]**| 0 - 1.0 |

## Injection Pulsers (INJ_OUT)

| Specification | Value |
| --- | --- |
| **# of Channels** |  4 |

**Notes:**
1. **Edge Timing:** The falling (high-to-low) edge transition time of injection pulses is measured to be ~ 6 nanoseconds, with the rising (low-to-high) transition time much larger. (Based on measurements at FNAL)

## Full-Duplex GTx Links

## LVDS Links (LVDS)

| Specification | Value |
| --- | --- |
| **# of Links (bidirectional)** |  17 |
| **Maximum bit rate ** | measured up to several hundred Mb/s (at DESY) |

## CMOS Outputs (CMOS_OUT)

Outputs from Caribou to the ASIC

| Specification | Value |
| --- | --- |
| **# of Channels** |  10 |
| **CMOS High Voltage [V]**| 0.8 - 3.6 (adjustable) |

## CMOS Inputs (CMOS_IN)

Inputs from the ASIC to Caribou

| Specification | Value |
| --- | --- |
| **# of Channels** |  14 |
| **CMOS High Voltage [V]**| 0.8 - 3.6 (adjustable) |

## Programmable Clock Generator

Si5345, programmable using ClockBuilder Pro + Peary.

Output Clocks 2 and 4 are routed to the SEARAY connector.
