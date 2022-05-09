# perky-janus
BeagleBone Black as a NTP server using GPS as a time source.

## Introduction
This project is to share my experience using a [BeagleBone Black](https://beagleboard.org/black) to obtain time from a GPS and then sharing time with other computers on a local network.  There are many stale posts about this topic, so here is a fresh take to throw on the pile.  I am writing this during May, 2022 for the BeagleBone Black (Revision C) using [AM3358 Debian 10.3 2020-04-06 4GB SD IoT](https://debian.beagleboard.org/images/bone-debian-10.3-iot-armhf-2020-04-06-4gb.img.xz)

The deployment domain is a network of single board computers in a mobile application.  Little computers have terrible clock drift, so an external source was a priority.  I also needed location (because, mobile) so a GPS was mandatory.

## The Plan
1. Purchase a GPS receiver w/PPS output (probably not USB)
1. Flash a BeagleBone Black w/a fresh image
1. Install software packages 
1. Configure GPS and PPS
1. Configure time server

Implementation follows, there is certainly room for substitution.

### GPS Receivers
I purchased several inexpensive USB GPS receivers via Amazon, which all delivered location data but would not drive ntpd(8), because they lacked a PPS output.

This [GPS Receiver](https://www.digikey.com/en/products/detail/adafruit-industries-llc/746/5353613?utm_adgroup=Essen%20Deinki&utm_source=google&utm_medium=cpc&utm_campaign=Shopping_DK%2BSupplier_Other&utm_term=&utm_content=Essen%20Deinki&gclid=Cj0KCQjw1N2TBhCOARIsAGVHQc5wzGDhJDlvyq4N77R9zlWtRVCpPK9Ajwizl2vyqLFRE6OX0z9Cs-8aAtAfEALw_wcB) was my solution (w/a powered antenna).

I used a [Digilent Digital Discovery](https://digilent.com/shop/digital-discovery-portable-usb-logic-analyzer-and-digital-pattern-generator/) to verify the GPS receiver was healthy.

![GPS output](https://github.com/guycole/perky-janus/blob/main/grafix/uart_out.png)
![PPS output](https://github.com/guycole/perky-janus/blob/main/grafix/pps_out.png)

### Install packages
1. apt-get install gps
1. apt-get install gpsd-clients
1. apt-get install pps-tools

### Configure GPS and PPS

The GPS receiver will write ASCII messages to BeagleBone UART1, and send a sync pulse to GPIO_60. gpsd(8) will readily accept the UART messages, but the pps driver will require some extra work.  

#### Pins
| Header | Pin | Name      | Description       |
|--------|-----|-----------|-------------------|
| P9     |  1  | DGND      | Ground            |
| P9     |  7  | SYS_5V    | GPS VIN           |
| P9     | 12  | GPIO_60   | GPS PPP           |
| P9     | 24  | UART1_TXD | GPS RX            |
| P9     | 26  | UART1_RXD | GPS TX            |

#### gpsd(8)
1. Copy [gpsd.default](https://github.com/guycole/perky-janus/blob/main/gpsd.default) to /etc/default/gpsd
1. Start gpsd(8) by invoking ```systemctl restart gpsd.service```
1. Verify gpsd(8) by invoking ```systemctl status gpsd.service```
![resultsl](https://github.com/guycole/perky-janus/blob/main/grafix/systemctl.png)

1. Assuming gpsd(8) started, try your luck with cgps(1).
![resultsl](https://github.com/guycole/perky-janus/blob/main/grafix/cgps.png)

XOXOXOXO

## BeagleBone History
Keep the below in mind while reading some of the older posts on about using GPS w/your BeagleBone.

[Quote](https://elinux.org/Beagleboard:BeagleBoneBlack_Debian#U-Boot_Overlays)
> Kernel Overlays are going bye-bye, too many bugs, too many race conditions, no kernel maintainers interested. We've said our farewells and U-Boot Overlays is the way forward: https://elinux.org/Beagleboard:BeagleBoneBlack_Debian#U-Boot_Overlays With multiple parties working on the U-Boot infrastructure. 


https://www.kernel.org/doc/Documentation/pps/pps.txt

pps_ldisc wants DCD change



I have since learned that USB GPS receivers would not readily interface with ntpd(8) or chronyd(8), because the kernel driver needs PPS.  

pps device?  for kernel?

/dev/pps0 when gpsd starts

cgps not working

ppsteset /dev/pps0
ppstest /dev/pps1 works... who created this device

## Pins
    | Header | Pin | Name      | Description       |
    |--------|-----|-----------|-------------------|
    | P8     |  1  | GPIO_99   | Widget Toggle     |
    | P9     |  1  | DGND      | Ground            |
    | P9     |  7  | SYS_5V    | GPS VIN           |
    | P9     | 12  | GPIO_60   | GPS PPP           |
    | P9     | 24  | UART1_TXD | GPS RX            |
    | P9     | 26  | UART1_RXD | GPS TX            |

I2C pins?

Device Tree Overlay
universal cape does not connect GPIO to kernel PPS driver.

no /sys/devices/platform/bone_capemgr

ls /sys/class/pps
ls /sys/class/pps/pps0

/dev/tty01 = UART1 = ttyS1

pps0 says "/dev/ttyS1"
modprobe pps-ktimer
k, found 1 source(s), now start fetching data...
time_pps_fetch() error -1 (Connection timed out)
time_pps_fetch() error -1 (Connection timed out)

/etc/default/gpsd

