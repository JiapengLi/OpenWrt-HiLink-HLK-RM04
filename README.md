# OpenWrt-HiLink-HLK-RM04
Openwrt patch and installtion guide for HiLink HLK-RM04

## Introduction
HLK-RM04 is a small wifi module which is pruducted by HiLink,
it is based on Ralink RT5350 with some GPIOs raised out.  
HLK-RM04 has 4M flash and 16M SDRAM on chip, has 1 USB port, 2 Serial Ports(lite and full), 1 I2C, 2 GPIO(GPIO0, RIN). In different configure, you can get different number of GPIO, for example 8 GPIOs + 1 serial port.

## Files
                             
- **openwrt-add-support-for-hilink-hlk-rm04.patch** -- patch to add "HILINK HLK-RM04" to OpenWrt
- **openwrt-fix-enable-uartf-kernel-panic.patch** -- patch to fix the kernel panic after enable UARTF
- **uboot128.img** -- uboot for 16M SDRAM, if you don't modify the HLK-RM04, use this uboot image
- **uboot256.img** -- uboot for 32M SDRAM, some people may want to expand the Uboot, and these men should use this uboot image
- **[hlk-rm04-boot-log.md](./hlk-rm04-boot-log.md)** -- some hardware information and openwrt bootlog of HLK-RM04

## Patch and Compile Openwrt

In order to build OpenWrt for "HiLink HLK-RM04", you need to:

- download the latest OpenWrt trunk sources from svn
- download the patch
- apply the patch
- choose your target/subtarget/profile for the build
- compile the firmware 

This is achieved using the following code snippet:

    mkdir openwrt
    cd openwrt
    svn co svn://svn.openwrt.org/openwrt/trunk@37737 <--@37737 means force to check out Revision 37737
    git clone https://github.com/JiapengLi/OpenWrt-HiLink-HLK-RM04.git
    cd trunk
    patch -p0 <../OpenWrt-HiLink-HLK-RM04/openwrt-add-support-for-hilink-hlk-rm04.patch
    patch -p0 <../OpenWrt-HiLink-HLK-RM04/openwrt-fix-enable-uartf-kernel-panic.patch

Also you may want some extra package(if not, skip then) :

	./scripts/feeds update -a
	./scripts/feeds install -a

Then, empty the ./tmp and configure your openwrt. Run:
	rm -rf tmp
	make menuconfig

In the configuration menu, you need to select the following options:

Target System: Ralink RT288x/RT3xxx
Subtarget: RT305x based boards
Target Profile: HILINK HLK-RM04

To use OpenWrt with LuCI Web UI, you can additionally select following options:

- LuCI --> Collections --> luci
- LuCI --> Protocols --> luci-proto-3g

After all the needed options are selected, exit the menu, save the configuration, and proceed to build:

	make

After compiling is done without any error, you'll find the image in `bin/ramips/` which is named `openwrt-ramips-rt305x-hlk-rm04-squashfs-sysupgrade.bin`. 

## Installtion
So far, i don't make WEBUI upgrade firmware work for flash openwrt image. This installtion need two steps:

- replace the HiLink official uboot 
- use the new uboot to flash Openwrt image

### Prepare
- USB2Serial or USB2TTL tool
- One Network cable
- serial concole(Eg: putty)
- tftp server, (ubuntu tftp-hpa; Windows Tftpd32.exe)

### Step by Step

Replace the uboot

1. set your PC ipadress to **192.168.16.100**, Gateway **192.168.16.254**, may be difference if you have changed it once.
1. connect HLK-RM04 LAN port to your PC, power up HLK-RM04
1. access <http://192.168.16.254/adm/hlk_update_www_hlktech_com.asp>, replace `192.168.16.254` with yours, if you've changed it once.
1. in **Update Bootloader** option, click **Choose File**, then select **uboot128.img**, click **Apply** then click **OK** to make sure.
1. Here, we have replaced the uboot of HLK-RM04 successfully. And now the hlk-rm04 works fine, only thing what we change is let Uboot 'talk' to us, so that we can use uboot to download the openwrt image.

**Flash Openwrt with Uboot**

1. configure a tftp server. 
	- For Ubuntu, See more [Install tftp info][Install tftp]

	```bash
		sudo atp-get install tftpd-hpa 
		sudo service tftpd-hpa 
		cp bin/ramips/openwrt-ramips-rt305x-hlk-rm04-squashfs-sysupgrade.bin /var/lib/tftpboot/	
	```
 	- Download Tftpd32.exe, make `openwrt-ramips-rt305x-hlk-rm04-squashfs-sysupgrade.bin' to be in the same directory with Tftpd32.exe, choose a right server ip.
1. open serial use 57600,8,n,1, make sure you have connect the serial cable.
1. Restart your HLK-RM04 module. Push '2' rapidly to enter the tftp write flash mode.
1. Set device ip `192.168.16.1`, Set server ip `192.168.16.100`.
1. input the file name `openwrt-ramips-rt305x-hlk-rm04-squashfs-sysupgrade.bin`

	```
		2: System Load Linux Kernel then write to Flash via TFTP.
		 Warning!! Erase Linux in Flash then burn new one. Are you sure?(Y/N)
		 Please Input new ones /or Ctrl-C to discard
		        Input device IP (10.10.10.123) ==:192.168.16.1
		        Input server IP (10.10.10.3) ==:192.168.16.100
		        Input Linux Kernel filename (tim_uImage) ==:openwrt-ramips-rt305x-hlk-rm04-squashfs-sysupgrade.bin
	```
1. Wait flashing finished, then you can use Openwrt on HLK-RM04. 

## About Patch

### openwrt-add-support-for-hilink-hlk-rm04.patch
This patch is based on previous work by Squonk42 (<https://github.com/Squonk42/OpenWrt-RT5350>), shmygov (<https://github.com/shmygov/OpenWrt-HAME-MPR-A2>)  and others from OpenWrt forum (https://forum.openwrt.org/viewtopic.php?id=37002&p=19), adapted for the new "Device Trees" structure based on dts files used in the latest OpenWrt trunk. Because need to change the order of `uart` and `uartlite` node in the rt5350.dtsi file, so that linux kernel register `uartlite` and `uart` in `ttyS0` and `ttyS1` sequence, so didn't use the rt5350.dtsi file just create a new one.  
**Note** This patch use GPIO0 for system status LED, you may need solder one on you HLK-RM04. And WPS button is connected to "RIN/GPIO14" pin.

### openwrt-fix-enable-uartf-kernel-panic.patch
After enable uartf we come across a kernel panic, this patch fixes it. Reference [OpenWrt Ticket #13590][ticket]

## End
This patch is maked with the help of many guys, jeff who find the the extra at command, Tao Zhou who help to solve the ttyS* sequence problem, and John Crispin and Sebastian Muszynski from the openwrt mail list.

[Install tftp]: http://www.cyberciti.biz/faq/install-configure-tftp-server-ubuntu-debian-howto/
[ticket]: https://dev.openwrt.org/ticket/13590
