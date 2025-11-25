# Step 1: Install Python 3.11

**Windows:** Download and install Python from https://www.python.org/downloads/release/python-3114/

**Linux:** Install Python via your package manager if it is not already installed. NOTE: If you are using an old operating system (for example **Scientific Linux 7**) which does not have Python 3.11 available via package manager, you will need to build it from source. Once you have a Python binary built for your operating system, you can point your Spacely virtual environment to it in Step 4(a).

**Important Notes:**
1. Make sure that you are using 64-bit Python (not 32-bit),
2. If necessary, **edit the system PATH** to point to the correct version of Python.

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

# Step 4: Setup Spacely

Run the appropriate setup script based on your operating system (SetupWindows.ps1 or SetupLinux.sh). For typical use cases, the script will automatically carry out steps 4(a), 4(b), and 4(c) without user input. The following sections provide for some cases where manual intervention may be necessary.

## Step 4(a): Create your Virtual Environment 

As long as Python 3.11 is on the system path, the setup script will create a virtual environment in the *venv* folder by running ```python -m venv venv``` and activate it by running ```.\venv\Scripts\activate```.

**If you built your own Python from source in Step 1,** you should manually create the venv by running 

```
/path/to/your/python -m venv venv
```

## Step 4(b): Python Library Installation 

The setup script will download and install all the necessary python libraries which are listed in the /requirements/ folder. 

**If you are running on Windows and you encounter an error with permissions,** retry after running the following command:

```
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

**If you are running on Scientific Linux 7** or any computer using an old version of GCC, you may not be able to successfully build matplotlib. You will need to independently build a matplotlib wheel for your operating system and then install it by running 
```
pip install "matplotlib==3.7.2" --only-binary=:all:
```

## Step 4(c): Installation of py-libs-common

The setup script will download and install all the py-libs-common libraries needed to run Spacely. 

**If you are a Spacely developer,** it is recommended to make an *editable* installation of py-libs-common libraries. This essentially means that you will link Spacely to your local copy of the py-libs-common source code, so that any changes in that source code will be immediately reflected the next time you run Spacely. 

For each of the following subdirectories of py-libs-common, run "pip install -e <directory>" where <directory> is the path to where your local copy of that directory is stored:

- /py-libs-common/libinstrument/
- /py-libs-common/liblogwizard/
- /py-libs-common/libIO/
- /py-libs-common/nitoolbox/


If you ever need to reinstall a particular python library, use this command:
```
pip install --force-reinstall --no-deps <library>
```

# Step 5: (NI-PXI Only) NI Driver Installation 

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
