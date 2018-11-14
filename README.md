# WAVE300
Sources for Lantiq WAVE300 wi-fi driver used in many embedded devices.

## Scope
In this document are instructions in order to setup and run
the Lantiq Wave 300 development kit that includes drivers and misc
tools.

The aim of this project is to upbring the sources to newer Linux kernels 
as original sources available seem to be compatible only with older kernels.

## LEGAL DISCLAIMER
The original sources for this git (wave300-3.2.1.1.48.tar.gz), upon which 
all changes are made, came with no license file. In the code, however
the MODULE_LICENSE is clearly defined as "proprietary".

In previous releases of the drivers like 3.01.0, there's a GPL 
LICENSE (txt) file where dual BSD/GPL is granted.

Being the sources of this git an evolution of the old ones, 
the GPL LICENSE (txt) file and its content must be kept.

Nonetheless, being myself an absolute newcomer into the world of
open-source and licensing, I beg the original authors & owners of this 
code Infineon/Lantiq/Intel to correct me if I'm wrong and I will 
immediatly turn down this git.

## Status
- The code has been up-ported to latest OpenWRT trunk using kernel 4.9 or later
- The code was successfully compiled using Ubuntu 16.04 and Ubuntu 18.04 on WSL (Windows Subsystem for Linux)
- The compiled kernel module is successfully loaded, and network device initiated.
- The driver communicates with a custom hostapd that contains mtlk primitives.
- There is no IP communication, this is likely related to "SOURCE of RX packet not found!" issue.

## TODO
- Diagnose "SOURCE of RX packet not found!" issue which is likely why there is no IP connectivity.


## HOWTO: setup build environment
* Build using OpenWRT 18.06.1 on Ubuntu 16.04 LTS or Ubuntu 18.04 for WSL.


### Clone the OpenWRT development environment
`mkdir ~/src`  
`cd ~/src`  
`git clone https://github.com/openwrt/openwrt.git`  
It is better to switch to current stable release.  
`git checkout -f v18.06.1`  
`cd openwrt`  

### Select target and subtarget
`make menuconfig`  

### Build OpenWRT
This step is necessary to populate toolchain and kernel directories  
`make`  

### Install libnl in OpenWRT
`./scripts/feeds update -a`  
`./scripts/feeds install libnl`  
`make package/libnl/{clean,compile} V=99`  

Note: libnl-tiny does not work by default

I also added the following options to my .config :

For lspci:
`CONFIG_PACKAGE_pciutils=y`  

I like vim:
`CONFIG_PACKAGE_vim-full=y`  

For iwconfig:
`CONFIG_PACKAGE_wireless-tools=y`  

For hostapd:
`CONFIG_PACKAGE_libopenssl=y` 


## HOWTO: Build driver
We are now ready to build the driver.

### Clone WAVE300 git repository
`cd ~/src`  
`git clone https://github.com/vittorio88/WAVE300.git`  
`cd WAVE300/driver`  

### Set variable to point to cloned OpenWRT SDK repository
`export BSP_BUILD_ROOT=~/src/openwrt`  

### Configure build
`make menuconfig`  

You should see the classical blue screen menu.  
1. Choose your target platform<sup>1</sup>  

<sup>1</sup> ugw5.1-vrx288 has been more thoroughly tested and will be used as example for the rest of the document.  

Upon exiting make menuconfig, ./configure will be run automatically. Do not run it on your own.  
Note: If this step fails, then maybe you have multiple toolchains in openwrt/staging_dir. Remove all but the latest.

### Build driver
`make`  

Drivers will be available at:  
`./builds/ugw5.1-vrx288/binaries/wls/driver/mtlk.ko`  
`./builds/ugw5.1-vrx288/binaries/wls/driver/mtlkroot.ko`  


## HOWTO: Deploy driver

### Prepare firmware files
In order to run the wifi driver in your target machine
you need to have the following firmware files:  
`/lib/firmware/sta_upper.bin`  
`/lib/firmware/ap_upper.bin`  
`/lib/firmware/contr_lm.bin`  
`/lib/firmware/Progmodel_*.bin`  

Check README_FW in `/lantiq_fw` for help in obtaining these files.
They may need to be renamed, you will see in kernel log if it doesn't find an
adequate firmware image.

## Load driver
Bring those files to your platform (router, embedded system, etc) and
insert the modules (insmod) in the following order:  
`insmod mtlkroot.ko`  
`insmod mtlk.ko ap=1`  

A new wifi interface should be visible with iwconfig, usually named wlan0.  
`iwconfig`  

## Configure wireless interface
Below lines are an example if your radio is 2.4GHz:  
`iwpriv wlan0 sCountry IT`  
`iwconfig wlan0 essid test-24ghz`  
`iwconfig wlan0 channel 1`  

Below lines are an example if your radio is 5GHz:  
`iwpriv wlan0 sCountry GB`  
`iwconfig wlan0 essid test-5ghz`  
`iwconfig wlan0 channel 52`  

Finally bring up the interface:  
`ifconfig wlan0 up`  

Here is it possible to see the wireless network and connect, yet 
there is no IP connectivity.

It is also possible to use hostapd if patched with mtlk primitives.

## Troubleshooting
The generated modules accept some parameters, most of them are unexplored.

### Increase logging verbosity
At menuconfig level, set debug level, 9 being the highest.  
Beware, because the size of the drivers will grow.

### Insert driver(s) with increased logging verbosity
`insmod mtlkroot.ko cdebug=3`

