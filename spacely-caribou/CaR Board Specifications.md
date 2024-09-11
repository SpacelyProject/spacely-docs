# CaR Board Specifications


## Power Supplies

| Specification | Value |
| --- | --- |
| **# of Channels** |  8 |
| **Voltage Range [V]**| 0.8 - 3.6 |


## Voltage References

| Specification | Value |
| --- | --- |
| **# of Channels** |  32 |
| **Voltage Range [V]**| 0 - 4 |

## Current References

| Specification | Value |
| --- | --- |
| **# of Channels** |  8 |
| **Current Range [A]**| 0 - 0.001 |

## Slow ADC Channels

| Specification | Value |
| --- | --- |
| **# of Channels** |  8 |
| ** Component** | ADS7828 |
| **Voltage Range [V]**| 0 - 4 |
| **Input Capacitance**| ~ 1 nF |

Note: The input capacitance of slow ADC channels is dominated by a 1nF filter cap on the CaR board. The input capacitance of the ADS7828 alone is 25 pF. 


## Fast ADC Channels

| Specification | Value |
| --- | --- |
| **# of Channels** |  16 |
| **Voltage Range [V]**| 0 - 1.0 |

## Injection Pulsers

## Full-Duplex GTx Links

## LVDS Links

## CMOS Outputs

| Specification | Value |
| --- | --- |
| **# of Channels** |  10 |
| **CMOS High Voltage [V]**| 0.8 - 3.6 (adjustable) |

## CMOS Inputs

| Specification | Value |
| --- | --- |
| **# of Channels** |  14 |
| **CMOS High Voltage [V]**| 0.8 - 3.6 (adjustable) |

## Programmable Clock Generator

Si5345, programmable using ClockBuilder Pro + Peary.