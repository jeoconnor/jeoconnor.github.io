# raspberry-pi-network-boot
How to boot a raspberry pi over a network.

To do this involves setting up an install server subnet from a
separate host computer.  In this case I'm using my desktop running
LinuxMint 19.3.

## Server configuration
On the server we'll need to set up both a DHCP and a TFTP server to
handle network requests on the subnet.

In establishing that the DHCP server is working correctly I used a
standard Raspian install to see the DHCP server as a first step prior
to involving the TFTP server.
### Setup subnet

My home router subnet is 192.168.1.0/24 so I'm choosing
192.168.10.0/24 for the install subnet.  For the subnet network I'm
using a USB ethernet adapter which shows up on my system as:
```
5: enx681ca2130a63: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 68:1c:a2:13:0a:63 brd ff:ff:ff:ff:ff:ff
```
Configure `enx681ca2130a63` to have a static IP of 192.168.10.1 and a netmask 255.255.255.0.

### Install DHCP server 
```
$ sudo apt install isc-dhcp-server
```
First add the following content to `/etc/dhcp/dhcpd.conf`.

In LinuxMint the ethernet adapter that the dhcp server will use is defined in `/etc/default/isc-dhcp-server`.  Add or edit the folling lines for the ethernet interface listed above:
```
INTERFACESv4="enx681ca2130a63"
INTERFACESv6="enx681ca2130a63"
```

If you want a static IP aasigned to the RPi uncomment out the two lines below.

```
$ cat /etc/dhcp/dhcpd.conf
max-lease-time 86400;

authoritative;

log-facility local7;

subnet 192.168.10.0 netmask 255.255.255.0 {
  range 192.168.10.101 192.168.10.200;

  option domain-name "home.oconnor.io";
  option domain-name-servers 192.168.10.1, 8.8.8.8, 8.8.4.4;
  option routers 192.168.10.1;

  host raspberry {
#    hardware ethernet b8:27:eb:b1:14:9b;
#    fixed-address 192.168.10.100;
    next-server 192.168.10.1;
    option tftp-server-name "192.168.10.1";
  }

}
```
The `next-server` option specifies the TFTP server.

Boot/reboot the raspberry pi and check that `eth0` is now configured
on the `192.168.10.0/24` subnet. Eg.,
```
pi@raspberrypi:~$ ip addr show dev eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether b8:27:eb:b1:14:9b brd ff:ff:ff:ff:ff:ff
    inet 192.168.10.117/24 brd 192.168.10.255 scope global dynamic noprefixroute eth0
       valid_lft 403sec preferred_lft 328sec
    inet6 fe80::79ba:f7db:a398:84af/64 scope link 
       valid_lft forever preferred_lft forever
```

### Install TFTP server
```
$ sudo apt install atftpd
```
In the configuration file: `/etc/default/atftpd` set the two options:
```
USE_INETD=false
OPTIONS="/tftpboot"
```
Create the directory `/tftpboot` and make it world writable and owned by `nobody`.
```
$ sudo mkdir /tftpboot
$ sudo chown nobody /tftpboot
$ sudo chmod 777 /tftpboot
```

Since we're not using `inetd` or one of its clones, start the `atfptd` server with the command,
```
$ sudo /etc/init.d/atftpd start
```
Populate the `/tftpboot` directory with the files, `bootcode.bin`,
`fixup.dat` and `start.elf` which can be taken from the `/boot`
partition of a Raspbian` SD card install.
### Serial Port Console

Add the entry, `enable_uart=1` to the file `/boot/config.txt` on an SD card set up to boot a pi.

Default baud rate is 115200.

For using minicom on the host to connect to the serial port the configuration settings were:
```
    +-----------------------------------------------------------------------+   
    | A -    Serial Device      : /dev/ttyUSB0                              |
    | B - Lockfile Location     : /var/lock                                 |   
    | C -   Callin Program      :                                           |   
    | D -  Callout Program      :                                           |   
    | E -    Bps/Par/Bits       : 115200 8N1                                |
    | F - Hardware Flow Control : No                                        |   
    | G - Software Flow Control : Yes                                       |
    |                                                                       |   
    |    Change which setting?                                              |
    +-----------------------------------------------------------------------+   
```

Note the hardware and software flow control settings.  These are
reversed from the minicom defaults.


### Enable network boot
In the /boot/config.txt file of a full Raspbian install add the entry,
```
program_usb_boot_mode=1
```
And boot.  After booting, to verify that the change took effect, 
```
$ vcgencmd otp_dump | grep 17:
17:3020000a
``` 
The `3020000a` is what to expect.

Now remove the SD card from the Raspberry PI and powercycle the Raspberry PI.
`/var/log/syslog` will show tftp messages: 
```
Jun 25 13:00:32 maxwell dhcpd[2048]: DHCPDISCOVER from b8:27:eb:b1:14:9b via enp4s0
Jun 25 13:00:32 maxwell dhcpd[2048]: DHCPOFFER on 192.168.10.100 to b8:27:eb:b1:14:9b via enp4s0
Jun 25 13:00:34 maxwell atftpd[2287]: Serving bootcode.bin to 192.168.10.100:49152
Jun 25 13:00:34 maxwell ntpd[2796]: Listen normally on 61 enp4s0 192.168.10.1:123
Jun 25 13:00:34 maxwell ntpd[2796]: Listen normally on 62 enp4s0 [fe80::748f:640b:9d8b:99a%2]:123
Jun 25 13:00:34 maxwell ntpd[2796]: new interface(s) found: waking up resolver
Jun 25 13:00:34 maxwell atftpd[2287]: Serving bootsig.bin to 192.168.10.100:49152
Jun 25 13:00:34 maxwell dhcpd[2048]: DHCPDISCOVER from b8:27:eb:b1:14:9b via enp4s0
Jun 25 13:00:34 maxwell dhcpd[2048]: DHCPOFFER on 192.168.10.100 to b8:27:eb:b1:14:9b via enp4s0
Jun 25 13:00:34 maxwell atftpd[2287]: Serving 2bb1149b/start.elf to 192.168.10.100:49153
Jun 25 13:00:34 maxwell atftpd[2287]: Serving autoboot.txt to 192.168.10.100:49154
Jun 25 13:00:34 maxwell atftpd[2287]: Serving config.txt to 192.168.10.100:49155
Jun 25 13:00:34 maxwell atftpd[2287]: Serving recovery.elf to 192.168.10.100:49156
Jun 25 13:00:34 maxwell atftpd[2287]: Serving start.elf to 192.168.10.100:49157
Jun 25 13:00:35 maxwell atftpd[2287]: Serving fixup.dat to 192.168.10.100:49158
Jun 25 13:00:35 maxwell atftpd[2287]: Serving recovery.elf to 192.168.10.100:49153
Jun 25 13:00:35 maxwell atftpd[2287]: Serving config.txt to 192.168.10.100:49154
Jun 25 13:00:35 maxwell atftpd[2287]: Serving dt-blob.bin to 192.168.10.100:49155
Jun 25 13:00:36 maxwell atftpd[2287]: Serving recovery.elf to 192.168.10.100:49156
Jun 25 13:00:36 maxwell atftpd[2287]: Serving config.txt to 192.168.10.100:49157
Jun 25 13:00:36 maxwell atftpd[2287]: Serving bootcfg.txt to 192.168.10.100:49158
Jun 25 13:00:36 maxwell atftpd[2287]: Serving bcm2710-rpi-3-b.dtb to 192.168.10.100:49159
Jun 25 13:00:36 maxwell atftpd[2287]: Serving cmdline.txt to 192.168.10.100:49160
Jun 25 13:00:36 maxwell atftpd[2287]: Serving recovery8.img to 192.168.10.100:49161
Jun 25 13:00:36 maxwell atftpd[2287]: Serving recovery8-32.img to 192.168.10.100:49162
Jun 25 13:00:36 maxwell atftpd[2287]: Serving recovery7.img to 192.168.10.100:49163
Jun 25 13:00:36 maxwell atftpd[2287]: Serving recovery.img to 192.168.10.100:49164
Jun 25 13:00:36 maxwell atftpd[2287]: Serving kernel8.img to 192.168.10.100:49165
Jun 25 13:00:36 maxwell atftpd[2287]: Serving kernel8-32.img to 192.168.10.100:49166
Jun 25 13:00:36 maxwell atftpd[2287]: Serving kernel7.img to 192.168.10.100:49167
Jun 25 13:00:36 maxwell atftpd[2287]: Serving kernel.img to 192.168.10.100:49168
Jun 25 13:00:36 maxwell atftpd[2287]: Serving armstub8.bin to 192.168.10.100:49169
Jun 25 13:00:36 maxwell atftpd[2287]: Serving kernel8.img to 192.168.10.100:49170
```

Looks good.
