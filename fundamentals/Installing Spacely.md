# Step 1: Install Python 3.11
https://www.python.org/downloads/release/python-3114/

WARNING: Make sure that you are using 64-bit Python (not 32-bit),

If necessary, **edit the system PATH** to point to the correct version of Python. 


# Step 2: Install Git 

## Installing Git on Windows

Download Github Desktop: https://desktop.github.com/

## Installing Git on Linux

Install Git through your package manager.

It is recommended to use an SSH key for authentication to Github.

https://docs.github.com/en/authentication /connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent

If you are connecting to a new repo using “git remote add”, make sure to always add the SSH version of the repo name, which should start with “git@github.com” rather than the HTTPS version of the name, which starts with “https”. This will allow you to authenticate using SSH key only rather than having to enter username + pwd every time.


# Step 3: Clone Spacely Repos

First, clone the core Spacely repository from https://github.com/SpacelyProject/spacely

The Spacely repo contains a subfolder /spacely-asic-config/ which is intended to contain all ASIC-specific configuration and routine files. We recommend creating a Git repository to track changes to these files, which is independent of the Spacely core repository. 

If you are a spacely developer, it is also recommended to clone the "py-libs-common" repository from https://github.com/SpacelyProject/spacely

If you will develop firmware for a Spacely-Caribou project, it is also recommended to clone the "spacely-caribou-common-blocks" directory, which can be used as a resource for your firmware development.


# Step 4: Library Installation 

## Python Library Installation 

Run the appropriate setup script based on your operating system (SetupWindows.ps1 or SetupLinux.sh). This script will activate a virtual environment, then download and install all the necessary libraries which are listed in the /requirements/ folder. 

**Note:** If you are running on Windows, and you encounter an error with permissions, retry after running the following command:

```
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

## Editable installation of py-libs-common

If you are a Spacely developer, it is recommended to make an editable installation of py-libs-common libraries. This essentially means that you will link Spacely to your local copy of the py-libs-common source code, so that any changes in that source code will be immediately reflected the next time you run Spacely. 

For each of the following subdirectories of py-libs-common, run "pip install -e <directory>" where <directory> is the path to where your local copy of that directory is stored:

- /py-libs-common/libinstrument/
- /py-libs-common/liblogwizard/
- /py-libs-common/libIO/
- /py-libs-common/nitoolbox/


If you ever need to reinstall a particular python library, use this command:
```
pip install --force-reinstall --no-deps <library>
```

## NI Driver Installation 

If you are using the NI-PXI system (not Caribou), ensure that up-to-date versions of the following drivers are installed:

- FlexRIO
- NI-DCPower


# (Optional) Recommendations for Ease of Use

On Windows, Spacely looks best when run with Windows Terminal, which may be installed from the Microsoft App Store.

On Windows, you can use the following steps to create a convenient Taskbar Shortcut to run Spacely:

1.	RMB > Create New Shortcut on your desktop.
2.	For the shortcut path, enter the following, replacing the path to Spacely with the path to your install.
```
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -ExecutionPolicy Bypass -NoExit -File "C:\path\to\your\spacely\PySpacely\Spacely.ps1"
```
3.	From Windows Terminal settings, set Windows Terminal as the default terminal application. 
4. Change the icon of your shortcut to spacely_icon.ico from the main Spacely repository.
5.	RMB on your new shortcut > Pin to task bar.
