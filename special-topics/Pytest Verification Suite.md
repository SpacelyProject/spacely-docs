# Pytest Verification Suite 

Spacely uses Pytest to ensure code quality. Before pushing an update to Spacely, all existing checks should pass, and new checks should be added for any newly-implemented features. 

To run checks:
1. Activate the Spacely virtual environment (**venv\Scripts\activate**)
2. For **spacely**: Navigate to *spacely/PySpacely* and run **pytest -v**
3. For **py-libs-common**: Navigate to the top level directory (*py-libs-common*) and run **pytest -v**.

## Note on Spacely-Cocotb Tests

By default, the Spacely pytest suite includes tests for Spacely-Cocotb. These tests require the user to take extra steps to provide Spacely with the Verilog simulators and library paths that it need. They **will not pass** on the first attempt, but will instead throw "User Configuration Needed" errors that instruct the user on what to provide. For more information, see [Spacely-Cocotb Example Design](</digital-twin/spacely-cocotb-example-design.md>).

To avoid running Spacely-Cocotb tests, simply run:
```
pytest -v --ignore=test/Spacely_Cocotb_test.py
```