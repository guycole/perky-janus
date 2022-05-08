# perky-janus
BeagleBone Black as a NTP server using GPS as a time source.

## Introduction
This project is to share my experience using a [BeagleBone Black](https://beagleboard.org/black) to obtain time from a GPS and then sharing time with other computers on a local network.  There are many stale posts about this topic, so here is a fresh take to throw on the pile.  I am writing this during May, 2022 for the BeagleBone Black (Revision C) using [AM3358 Debian 10.3 2020-04-06 4GB SD IoT](https://debian.beagleboard.org/images/bone-debian-10.3-iot-armhf-2020-04-06-4gb.img.xz)

My context is a network of single board computers in a mobile application.  Little computers have terrible clock drift, so an external source was a priority.  I also needed location (because, mobile) so a GPS mandatory.

## History
[Quote](https://elinux.org/Beagleboard:BeagleBoneBlack_Debian#U-Boot_Overlays)
> Kernel Overlays are going bye-bye, too many bugs, too many race conditions, no kernel maintainers interested. We've said our farewells and U-Boot Overlays is the way forward: https://elinux.org/Beagleboard:BeagleBoneBlack_Debian#U-Boot_Overlays With multiple parties working on the U-Boot infrastructure. 

##
apt-get install gpsd and clients
apt-get install pps-tools

## GPS Receivers
I purchased several inexpensive USB GPS receivers via Amazon, which all worked for location but would not drive ntpd(8).  

[USB GPS Receiver](https://www.digikey.com/en/products/detail/adafruit-industries-llc/746/5353613?utm_adgroup=Essen%20Deinki&utm_source=google&utm_medium=cpc&utm_campaign=Shopping_DK%2BSupplier_Other&utm_term=&utm_content=Essen%20Deinki&gclid=Cj0KCQjw1N2TBhCOARIsAGVHQc5wzGDhJDlvyq4N77R9zlWtRVCpPK9Ajwizl2vyqLFRE6OX0z9Cs-8aAtAfEALw_wcB)

USB GPS receivers



I have since learned that USB GPS receivers would not readily interface with ntpd(8) or chronyd(8), because the kernel driver needs PPS.  

pps device?  for kernel?

/dev/pps0 when gpsd starts

cgps not working

ppsteset /dev/pps0

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

