---
title: "Aruba Convert AP to IAP"
date: 2023-02-02T14:21:03+01:00
draft: false
tags:
- Aruba
---

Aruba Campus AP's can only run with an Aruba Moblility Controller, this controller can be embedded or virtual. Luckily for us Aruba let us convert these AP's to InstantAP's (IAP). you can do it with the mobility controller or through the below method which doenst' require us to download the MVC and apply for a free license.. that process is to complicated.

## What is needed?
1) Serial connector (we'll use an RPI for this)
2) Jumper wires
3) Firmware files for your AP (does require a valid Aruba account on [https://asp.arubanetworks.com](https://asp.arubanetworks.com))
4) TFTP server (atftpd)

### Make serial connection
As I did not have a fancy USB to serial convertor I'll be using an old raspberryPi for this purpose. the AP-305 does not have an USB serial port, but uses 4 pins.
You'll need to boot the PI and connect with SSH. Run following command:
```
sudo raspi-config
--> Interface Options
--> Serial
--> Would you like a login shell? > NO (enter)
--> Would you like serial port hardware? > YES (enter)
sudo reboot
```

When the RPI is booted we can connect our jumper wires to the AP-305 and the GPIO pins. Make sure that you cross Tx with Rx to be able to communicate.
the AP-305 PIN layout:  
![AP-305 Console](/posts_images/aruba_ap305_console.png)

Google your RPI GPIO pin layout as not all the RPI's have the same layout and connect the jumper wires.


### Flashing procedure
Make a SSH connection to your RPI and connect to the serial port:
```
screen /dev/ttyAMA0 9600
```
(find your tty device by running: dmesg|grep tty)

Now connect the AP with your PoE switch and the console should show the boot procedure.
Stop the apboot by any key when you see this message **"Hit <Enter> to stop autoboot:3"**

Now run the following command's:

1) **proginv system ccode CCODE-RW-de6fdb363ff04c13ee261ec04fbb01bdd482d1cd**  
(Important, to write the country code. When converting to IAP the IAP must have country code to operate. RW means RestofWorld, the hash is an SHA1 hash with syntax: "RW-AP S/N", you can use following website to create the SHA1 hash: [http://www.sha1-online.com](http://www.sha1-online.com/))
2) **invent -w**  
(Important, converting AP into IAP)
3) **dhcp**  
(Get the ip address for the AP before upgrading os through TFTP.
If could not get the ip from DHCP server, use this command to set static ip for the ap "setenv ipaddr 192.168.1.2")
4) **setenv serverip 192.168.178.84**  
(Set TFTP server ip before flashing the firmware OS)
5) **upgrade os 0 ArubaInstant_xxx_6.x.x.x-4.x.x.x_5xxxx** 
6) **upgrade os 1 ArubaInstant_xxx_6.x.x.x-4.x.x.x_5xxxx**  
(there are 2 boot image on the ap, just to make sure it will not rollback to AP-OS)
7) **factory_reset**  
(optional, clean up the static ip that set before)
8) **saveenv**  
(IMPORTANT, save the configuration so the "turn off mobility" will not appear)
9) **reset**  

Please read the release notes, if the current firmware is very old you'll need an additional upgrade step with firmware 6.5.4 before flashing to 8.x.
You can upgrade from console to 6.5.4 firmware, after that login in to the webportal with credentials admin/admin. You can then proceed with flashing to the latest firmware from within the webportal.
Firmware above 8.x --> default credentials: admin/SN (serialnumber).


