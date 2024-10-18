# Pytest Verification Suite 

Spacely uses Pytest to ensure code quality. Before pushing an update to Spacely, all existing checks should pass, and new checks should be added for any newly-implemented features. 

To run checks:
1. Activate the Spacely virtual environment (**venv\Scripts\activate**)
2. For **spacely**: Navigate to *spacely/PySpacely* and run **pytest -v**
3. For **py-libs-common**: Navigate to the top level directory (*py-libs-common*) and run **pytest -v**.

The current verification suite focuses on GlueConverter and Spacely-Caribou methods; others will be added later.