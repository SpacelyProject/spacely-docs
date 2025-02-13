# Spacely Release Notes

### v0.1.0-alpha

- Released 10/18/2024
- First semantic version. 
- Initial Pytest suite for spacely and py-libs-common repositories.
- Unified PatternRunner architecture for both NI-PXI and Spacely-Caribou.



### v0.2.0

**py-libs-common version:** v0.1.0-alpha
**spacely-caribou-common-blocks version:** v0.1.0

- Released on 2/11/2025
- Standardized the process of sharing global variables among modules using Spacely Globals (sg). In addition, global configuration is now loaded from Master_Config.txt instead of directly from the source code of Master_Config.py. See [Global Variables and Configuration in Spacely](</spacely-caribou/Global Varialbes and Configuration in Spacely.md>) for details.
- Implemented a pytest suite for the Spacely-Caribou flow, and cleaned up that flow. See [Spacely Cocotb Example Design](</digital-twin/spacely-cocotb-example-design.md>)