# CaR Board Specifications


## Power Supplies (PWR_OUT)

| Specification | Value |
| --- | --- |
| **# of Channels** |  8 |
| **Voltage Range [V]**| 0.8 - 3.6 |

**Note:** The power supplies measure current with a 0.01 ohm feedback resistor, and the monitoring component has an input offset of up to 10 uV, which may cause significant issues when measuring currents of ~ a few mA or smaller. See https://www.ti.com/product/INA226. One solution is to swap the feedback resistor for a higher value. 


## Voltage References (BIAS)

| Specification | Value |
| --- | --- |
| **# of Channels** |  32 |
| **Component** | DAC7678 |
| **Voltage Range [V]**| 0 - 4 |
| **Short Circuit Output Current [mA]**| 25 |

## Current References (CUR)

| Specification | Value |
| --- | --- |
| **# of Channels** |  8 |
| **Current Range [A]**| 0 - 0.001 |

## Slow ADC Channels (VOL_IN)

| Specification | Value |
| --- | --- |
| **# of Channels** |  8 |
| **Component** | ADS7828 |
| **Voltage Range [V]**| 0 - 4 |
| **Input Capacitance**| ~ 1 nF |

Note: The input capacitance of slow ADC channels is dominated by a 1nF filter cap on the CaR board. The input capacitance of the ADS7828 alone is 25 pF. 


## Fast ADC Channels (ADC_IN)

| Specification | Value |
| --- | --- |
| **# of Channels** |  16 |
| **Voltage Range [V]**| 0 - 1.0 |

## Injection Pulsers (INJ_OUT)

## Full-Duplex GTx Links

## LVDS Links (LVDS)

## CMOS Outputs (CMOS_OUT)

| Specification | Value |
| --- | --- |
| **# of Channels** |  10 |
| **CMOS High Voltage [V]**| 0.8 - 3.6 (adjustable) |

## CMOS Inputs (CMOS_IN)

| Specification | Value |
| --- | --- |
| **# of Channels** |  14 |
| **CMOS High Voltage [V]**| 0.8 - 3.6 (adjustable) |

## Programmable Clock Generator

Si5345, programmable using ClockBuilder Pro + Peary.

Output Clocks 2 and 4 are routed to the SEARAY connector.
