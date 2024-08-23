# ZCU102 Setup for Spacely-Caribou

This guide will instruct you on how to boot the ZCU102 for Spacely-Caribou. Unless otherwise specified, all steps must be followed in order.

**Prerequisites:** You must have a host PC which you can use to communicate with the ZCU102. 


## Flash the Operating System 

**Skip this step if:** You already have an SD card with an operating system image in your ZCU102. 

**Prerequisites:** You must have a Petalinux Operating System image. If you do not have this, contact Spacely Developers. 


## Hardware Connections

Ensure that the following hardware connections are securely made:

### Connect CaR Board and DUT Board

If you are using a ZCU102 FMC Mezzanine board, connect the Mezzanine board to connectors HPC0 and HPC1 of the ZCU102. 

If you are using a Caribou Control-and-Readout (CaR) board, connect it to the FMC connector J2 on the Mezzanine board. If you do not have a Mezzanine board, connect the CaR board directly to the HPC0 connect of the ZCU102. (Without the Mezzanine, you will be able to use a limited subset of CaR board signals. TODO: Add link)

*Note:* The CaR board has jumpers which must be installed correctly. (TODO: CaR board setup)

If you have an ASIC device-under-test (DUT) board, connect that to the CaR board SEARAY connector J28.

### Connect ZCU102 to Host PC

Connect the ethernet, UART, and JTAG interfaces of the ZCU102 to your host PC

<p align="center">
<img src="https://github.com/SpacelyProject/spacely-docs/blob/main/figures/spacely-caribou/basic-setup/ZCU102_Serial_and_Ethernet_Connections.PNG" width="700">
</p>

## Power on the System

If you are using a CaR board, supply external power (12 V / 1 A) to the CaR board *before powering on the ZCU102.* This is critical so that the I2C multiplexers on the CaR board may be enumerated by the ZCU102 at boot time.

Ensure that ZCU102 SW6 is set to boot from SD.

<p align="center">
<img src="https://github.com/SpacelyProject/spacely-docs/blob/main/figures/spacely-caribou/basic-setup/ZCU102_SW6_Boot_from_SD.png" width="300">
</p>

Finally, power on the ZCU102. 


## Enable the ZCU102 Ethernet Connection 

When the ZCU102 boots, its ethernet interface. We must log into the ZCU102 using a serial console, such as minicom, in order to turn on this interface.

1. Ensure that you have **dialout** group permissions (or equivalent) to access the UART hardware interface.
2. Connect to the ZCU102 with minicom: **minicom -D /dev/ttyUSB0** (If ttyUSB0 does not work, your ZCU may be available on ttyUSB1.)
3. Open minicom serial port settings by pressing (Ctrl-A + O) and navigating to "Serial port setup". Ensure the following are set:
   1. Bps/Par/Bits = 115200 8N1
   2. Hardware Flow Control = No 
   3. Software Flow Control = No 
4. Press enter. A login prompt for the ZCU102 should appear. If it does not, then retry with a different /dev/ttyUSB device.
5. The login is "petalinux".
6. If this is your first time booting the system, you will be prompted to set a password. Otherwise, the password is whatever you set on the first time you booted the system. 
7. Once you have entered the password, you should have terminal access to the ZCU102. Enter the command **sudo ifconfig eth0 192.168.1.24** to enable the ethernet interface and set the ZCU102's IP address to 192.168.1.24
8. Exit minicom by pressing (Ctrl-A + X)
9. From the host PC, check that you were successful by attempting to SSH to the ZCU102: **ssh petalinux@192.168.1.24**
10. If this is not the first time you have booted petalinux, you may see a warning with the phrase "Remote Host Identification has Changed". If you see this, run the command **ssh-keygen -R 192.168.1.24** on the host PC. 


## Build Peary on the ZCU102

**Skip this step if:** You have already built peary on the ZCU102 in the past.


## Flash Firmware Image to the ZCU102

**Skip this step if:** You do not want to use the programmable logic (PL) side of the ZCU102. For example, you just want to check that linux / peary boots.

**Prerequisites:** You should have a firmware bitstream created in Vivado available on your host PC.

1. Ensure that you have **dialout** group permissions (or equivalent) to enable you to access the JTAG hardware interface. Warning: If you attempt this step without the correct permissions, you will not be able to see any targets in Vivado, and you will need to kill your hw_server instance.
2. Open Vivado Hardware Manager
3. Auto-connect or manually connect to your ZCU102.
4. Program the ZCU102 with your bitstream.

## Run Peary Server on the ZCU102 

1. SSH to the ZCU102. 
2. Navigate to your peary install directory (default: /home/petalinux/peary)
3. Run the command: **sudo ./bin/pearyd -v TRACE**  (The "-v TRACE" part is optional; it prints more debug information.)
