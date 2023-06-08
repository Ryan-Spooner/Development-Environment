# Notes
- Find out where these commands comes from:
```bash
# Command 1
arm-none-eabi-objcopy -O binary $1.elf $1.bin
```
```bash
# Command 2
sudo st-flash --reset write $1.bin 0x08000000
```
___

# USB/IP Client
*Connect a microcontroller to Windows Subsystem for Linux*
## Prerequisites
You should have WSL and Linux installed on your system: this tutorial uses a Windows 10 host and WSL2 with Ubuntu 22.04.2 LTS. However, it should work with other Debian-based distributions.

If you use a different package manager, etc., you can modify the code snippets to fit your environment. Check relevant documentation to inform your modifications.
## Turorial
### Step 1
Install usbipd using winget within PowerShell
```powershell
winget install usbipd
```
### Step 2
Verify your default WSL distribution
```powershell
wsl --list --all
```
```powershell
# Sample Output
Windows Subsystem for Linux Distributions:
Ubuntu-22.04
Ubuntu-20.04 (Default)
```
#### Step 2.1
If you want to change the default, call `wls --setdefault` against the preferred distribution's name as it appears in the list from the previous step
```powershell
wsl --setdefault Ubuntu-22.04
```
#### Step 2.2
Verify your selection by relisting the WSL distributions
```powershell
wsl --list --all
```
```powershell
# Sample Output
Windows Subsystem for Linux Distributions:
Ubuntu-22.04 (Default)
Ubuntu-20.04
```
The 'Default' indicator should now associate with your preferred distribution
### Step 3
==*(!) Do this step from within the WSL default distribution*==
Run the following commands so Linux can communicate via USB/IP
```bash
sudo apt install linux-tools-virtual hwdata
```
```bash
sudo update-alternatives --install /usr/local/bin/usbip usbip `ls /usr/lib/linux-tools/*/usbip | tail -n1` 20
```
### Step 4
==*(!) Switch back to PowerShell*==
Connect the microcontroller to your system's USB interface
### Step 5
List the USB devices recognized by your system
```powershell
usbipd wsl list
```
#### Step 5.1
Note the microcontroller's BUS ID
```powershell
> usbipd wsl list
BUSID  DEVICE                                      STATE
1-7    USB Input Device                            Not attached
4-4    STMicroelectronics ST-LINK/V2.1             Not attached
5-2    Surface Ethernet Adapter                    Not attached
```
### Step 6
Attach the microcontroller
```powershell
usbipd wsl attach --busid 4-4
```
### Step 7
Relist the connected USB devices to verify the microcontroller is attached
```poweshell
usbipd wsl list
```
Its state should have changed from 'Not attached' to 'Attached - <disto\>'
```powershell
> usbipd wsl list
BUSID  DEVICE                                      STATE
1-7    USB Input Device                            Not attached
4-4    STMicroelectronics ST-LINK/V2.1             Attached - Ubuntu
5-2    Surface Ethernet Adapter                    Not attached
```
### Step 8
==*(!) Switch back to WSL*==
List the attached USB devices
```bash
lsusb
```
The microcontroller should show up
```bash
$ lsusb
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 002: ID 0483:374b STMicroelectronics ST-LINK/V2.1
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```
If the microcontroller is visible, you can now interact with it using normal Linux tools
### Step 9
Detach the microcontroller from the Windows command line when you are finished
```powershell
usbipd wsl detach --busid 4-4
```
List the USB devices: Its state should now be 'Not attached'
```powershell
> usbipd wsl list
BUSID  DEVICE                                      STATE
1-7    USB Input Device                            Not attached
4-4    STMicroelectronics ST-LINK/V2.1             Not attached
5-2    Surface Ethernet Adapter                    Not attached
```
Sharing also stops if you unplug the microcontroller or restart your computer
___

# GNU ARM Toolchain
*draft*
Get the latest x86_64 Linux hosted cross toolchain from the [arm developer website](https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads)
![[arm_link.png]]
Right click the tarball link and paste it in a text editor. The link should look something like this:
```
https://developer.arm.com/-/media/Files/downloads/gnu/12.2.rel1/binrel/arm-gnu-toolchain-12.2.rel1-x86_64-arm-none-eabi.tar.xz?rev=71e595a1f2b6457bab9242bc4a40db90&hash=37B0C59767BAE297AEB8967E7C54705BAE9A4B95
```
Strip the query string from the link (everything after and including the "?" following `*tar.xz`). Your link should now look something like this:
```
https://developer.arm.com/-/media/Files/downloads/gnu/12.2.rel1/binrel/arm-gnu-toolchain-12.2.rel1-x86_64-arm-none-eabi.tar.xz
```
Download the file archive
```bash
curl -Lo gcc-arm-none-eabi.tar.xz "https://developer.arm.com/-/media/Files/downloads/gnu/12.2.rel1/binrel/arm-gnu-toolchain-12.2.rel1-x86_64-arm-none-eabi.tar.xz"
```
*Note: The `-o` flag writes to a file instead of stdout and `-L` allows curl to be redirected if the file is hosted somewhere unexpected*

Create a new directory to store the toolchain files, then extract the archive to it
```bash
sudo mkdir /opt/gcc-arm-none-eabi
```
```bash
sudo tar xf gcc-arm-none-eabi.tar.xz --strip-components=1 -C /opt/gcc-arm-none-eabi
```
Add the `/opt/gcc-arm-none-eabi/bin` directory to the PATH environment variable and append the `gcc-arm-none-eabi` script to `profile.d`
```bash
echo 'export PATH=$PATH:/opt/gcc-arm-none-eabi/bin' | sudo tee -a /etc/profile.d/gcc-arm-none-eabi.sh
```
Source the `profile` directory for the changes to take effect
```bash
source /etc/profile
```

___

# GNU Debugger
___

# Minicom
___
