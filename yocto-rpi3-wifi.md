# Building Yocto Image for Raspberry PI 3b with WIFI enabled

This page describes how to build a Yocto image for the Raspberry Pi 3b with wifi enabled.

## Create the workspace directory
```
$ mkdir ${TOPDIR}
$ cd ${TOPDIR}
```
## Check out the Yocto repositories
```
$ git clone git://git.yoctoproject.org/poky
$ git clone git://git.openembedded.org/meta-openembedded
$ git clone git://git.yoctoproject.org/meta-raspberrypi
```
## Set up environment for build directory

The directory name, `build` is arbitrary, but something like `build` or `build-rpi` helps to remember the purpose of the directory.
```
$ source poky/oe-init-build-env build
# Now in ${TOPDIR}/build
$ pwd
  "${TOPDIR}"/build
```
## Edit configuration files in the `conf` directory
Modify the `conf/bblayers.conf` file so that the `BBLAYERS` variable has the following definition:
```
BBLAYERS ?= " \
  ${BSPDIR}/poky/meta \
  ${BSPDIR}/poky/meta-poky \
  ${BSPDIR}/poky/meta-yocto-bsp \
  ${BSPDIR}/meta-openembedded/meta-oe \
  ${BSPDIR}/meta-raspberrypi \
  "
```

Modify the `conf/local.conf` file setting the following variables near the top of the file.

Set the machine type:
```
MACHINE = "raspberrypi3"
```

Optionally enable the serial console.  This requires a USB cable connected to some GPIO pins on the RPi -- not described here.
```
ENABLE_UART = "1"
```

Add an SSH server with `ssh-server-dropbear` and disable needing to supply a passwd to root when logging in with `debug-tweaks` (a development convenience).
``` 
EXTRA_IMAGE_FEATURES += "ssh-server-dropbear debug-tweaks"
```

```
IMAGE_INSTALL_append = " kernel-modules wireless-regdb-static linux-firmware-bcm43430 wpa-supplicant lshw"
```
`wireless-regdb-static` is needed by the `cfg80211` kernel module so it won't complain about not finding the `regulatory.db` file. 
`linux-firmware-bcm43430` is essential for enabling the wifi device on the RaspberryPI 3B.  I'm not sure if there might not be some variants that require different firmware, but the device was not detected by the driver, `brcmfmac`, without it. `wpa-supplicant` manages the wifi connection.
`lshw` isn't necessary but it is a convenient tool for checking whether the wifi device was detected by the OS.

After saving changes to these files switch back to the ${TOPDIR} directory.

## Build Yocto image

This will take several hours when run the first time.

```
$ bitbake core-image-minimal
```
When finished if no errors, you're ready to copy to a microsd.  I am building on LinuxMint 20.1 and get a warning about it being an unsupported OS, but I haven't had any problems so far.

## Copy image to MicroSD

I've been using `bmaptool` to write the microsd.  It is available on LinuxMint (and so I assume on Ubuntu) as the package `bmap-tools`. Needs to be executed as root.

Make sure to use the correct output device or you'll regret it.  One way to identify the correct device where the microsd is located is with the `lsblk` command. Executing `lsblk` *before* inserting the disk in my system yielded
```
$ lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0   1.8T  0 disk 
├─sda1   8:1    0   500M  0 part /boot/efi
├─sda2   8:2    0    40M  0 part 
├─sda3   8:3    0   128M  0 part 
├─sda4   8:4    0   500M  0 part 
├─sda5   8:5    0     1T  0 part 
├─sda6   8:6    0   450M  0 part 
├─sda7   8:7    0  12.7G  0 part 
└─sda8   8:8    0 781.3G  0 part /
sdb      8:16   0 372.6G  0 disk 
├─sdb1   8:17   0  14.9G  0 part [SWAP]
└─sdb2   8:18   0 357.7G  0 part 
sr0     11:0    1  1024M  0 rom  
$
```
After inserting the microsd yielded:
```
$ lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0   1.8T  0 disk 
├─sda1   8:1    0   500M  0 part /boot/efi
├─sda2   8:2    0    40M  0 part 
├─sda3   8:3    0   128M  0 part 
├─sda4   8:4    0   500M  0 part 
├─sda5   8:5    0     1T  0 part 
├─sda6   8:6    0   450M  0 part 
├─sda7   8:7    0  12.7G  0 part 
└─sda8   8:8    0 781.3G  0 part /
sdb      8:16   0 372.6G  0 disk 
├─sdb1   8:17   0  14.9G  0 part [SWAP]
└─sdb2   8:18   0 357.7G  0 part 
sdc      8:48   1  14.9G  0 disk 
├─sdc1   8:49   1  49.7M  0 part 
└─sdc2   8:50   1 140.4M  0 part 
sr0     11:0    1  1024M  0 rom  
```
So the microsd is device `sdc` here.

Switch to the directory containing the images:
```
$ cd tmp/deploy/images/raspberrypi3 
$ sudo bmaptool copy --bmap core-image-minimal-raspberrypi3.wic.bmap core-image-minimal-raspberrypi3.wic.bz2 /DEV/SDC
[sudo] password for oconnor:         
bmaptool: info: block map format version 2.0
bmaptool: info: 50279 blocks of size 4096 (196.4 MiB), mapped 28042 blocks (109.5 MiB or 55.8%)
bmaptool: info: copying image 'core-image-minimal-raspberrypi3.wic.bz2' to block device '/dev/sdd' using bmap file 'core-image-minimal-raspberrypi3.wic.bmap'
bmaptool: info: 100% copied
bmaptool: info: synchronizing '/dev/sdd'
bmaptool: info: copying time: 16.1s, copying speed 6.8 MiB/sec
$ 
```
Remove the microsd, insert into the RaspberryPi, and power it up.

## Login to RaspberryPi

As root, login to the RaspberryPI.
Check that wifi device was detected with `lshw`.  The following is the relevant portion extracted from the output of `lshw`. 
```
*-device
          description: SDIO Device
          physical id: 1
          bus info: mmc@1:0001
          serial: 0
          capabilities: sdio
        *-interface:0 DISABLED
             description: Wireless interface
             product: 43430
             vendor: Broadcom
             physical id: 1
             bus info: mmc@1:0001:1
             logical name: mmc1:0001:1
             logical name: wlan0
             serial: b8:27:eb:e4:41:ce
             capabilities: ethernet physical wireless
             configuration: broadcast=yes driver=brcmfmac driverversion=7.45.98.97 firmware=01-bf41ed64 multi1
        *-interface:1
             product: 43430
             vendor: Broadcom
             physical id: 2
             bus info: mmc@1:0001:2
             logical name: mmc1:0001:2
```
If the device isn't being correctly identified (here at `interface:0`) it will more likely show up as an entry analogous to the what's shown here as `interface:1`.

If the device shows up then confirm that it is not yet configured.
```
root@raspberrypi3:~# ip addr show dev wlan0
3: wlan0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop qlen 1000
    link/ether b8:27:eb:e4:41:ce brd ff:ff:ff:ff:ff:ff
root@raspberrypi3:~# 
```
## Configure wlan0 device
Edit `/etc/wpa_supplicant.conf` to have the following content:
```
ctrl_interface=/var/run/wpa_supplicant
ctrl_interface_group=0
update_config=1

network={
	scan_ssid=1
	ssid="SSID"
	psk="PASSPHRASE"
	key_mgmt=WPA-PSK
}
```
where `SSID` is the ssid for your wireless router and `PASSPHRASE` is the password.

Modify the `/etc/network/interfaces` file adding `wlan0` to the line containing `auto eth0`.  I.e., you want a line that looks like: 
```
auto eth0 wlan0
```

Now reboot the system.

After logging in again, check whether the `wlan0` device is configured. Now we should see:
```
root@raspberrypi3:~# ip addr show dev wlan0
3: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast qlen 1000
    link/ether b8:27:eb:e4:41:ce brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.97/24 brd 192.168.1.255 scope global wlan0
       valid_lft forever preferred_lft forever
    inet6 2601:283:487f:c753:ba27:ebff:fee4:41ce/64 scope global dynamic flags 100 
       valid_lft 592sec preferred_lft 592sec
    inet6 fe80::ba27:ebff:fee4:41ce/64 scope link 
       valid_lft forever preferred_lft forever
root@raspberrypi3:~# 
```

