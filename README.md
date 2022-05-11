# perky-janus
BeagleBone Black as a NTP server using GPS as a time source.

## Introduction
This project is to share my experience using a [BeagleBone Black](https://beagleboard.org/black) to obtain time from a GPS and then sharing time with other computers on a local network.  There are many stale posts about this topic, so here is a fresh take to throw on the pile.  I am writing this during May, 2022 for the BeagleBone Black (Revision C) using [AM3358 Debian 10.3 2020-04-06 4GB SD IoT](https://debian.beagleboard.org/images/bone-debian-10.3-iot-armhf-2020-04-06-4gb.img.xz)

The deployment domain is a network of single board computers in a mobile application.  Little computers have terrible clock drift, so an external source was a priority.  I also needed location (because, mobile) so a GPS was mandatory.

## The Plan
1. Purchase a GPS receiver w/PPS output (probably not a USB device)
1. Flash a BeagleBone Black w/a fresh image
1. Install software packages 
1. Configure GPS and PPS
1. Configure time server

Implementation follows, there is certainly room for substitution.

### GPS Receivers
I purchased several inexpensive USB GPS receivers via Amazon, which all delivered location data but would not drive ntpd(8), because they lacked a PPS output.

This [GPS Receiver](https://www.digikey.com/en/products/detail/adafruit-industries-llc/746/5353613?utm_adgroup=Essen%20Deinki&utm_source=google&utm_medium=cpc&utm_campaign=Shopping_DK%2BSupplier_Other&utm_term=&utm_content=Essen%20Deinki&gclid=Cj0KCQjw1N2TBhCOARIsAGVHQc5wzGDhJDlvyq4N77R9zlWtRVCpPK9Ajwizl2vyqLFRE6OX0z9Cs-8aAtAfEALw_wcB) was my solution (w/a powered antenna).

I used a [Digilent Digital Discovery](https://digilent.com/shop/digital-discovery-portable-usb-logic-analyzer-and-digital-pattern-generator/) to verify the GPS receiver was healthy.
1. GPS UART output
![GPS output](https://github.com/guycole/perky-janus/blob/main/grafix/uart_out2.png)
1. GPS PPS output
![PPS output](https://github.com/guycole/perky-janus/blob/main/grafix/pps_out2.png)

### Install packages
1. apt-get install gps
1. apt-get install gpsd-clients
1. apt-get install pps-tools
1. apt-get install ntp

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

#### pps
1. For reasons, pps will require you to install a device overlay.
1. git clone https://github.com/RobertCNelson/bb.org-overlays.git
1. Copy [BB-UART1-GPS-00A0.dts](https://github.com/guycole/perky-janus/blob/main/BB-UART1-GPS-00A0.dts) to bb.org-overlays/src/arm
1. Install the overlays by invoking bb.org-overlays/install.sh
1. Update /boot/uEnv.txt to reference the new overlay
![uEnv.txtl](https://github.com/guycole/perky-janus/blob/main/grafix/uenv.png)
1. Reboot
1. Test for PPS success
![ppstestl](https://github.com/guycole/perky-janus/blob/main/grafix/ppstest.png)
1. gpsmon(1) will demonstrate you have pps working
![resultsl](https://github.com/guycole/perky-janus/blob/main/grafix/gpsmon.png)
1. ntpshmmon(1) will also verify gpsd using PPS
![resultsl](https://github.com/guycole/perky-janus/blob/main/grafix/ntpshmmon.png)

### Configure Time Server
For my application, there are not internet connected time servers to consult.  I want time exclusively from the GPS receiver.
1. Copy [ntpd.conf](https://github.com/guycole/perky-janus/blob/main/ntpd.conf) to /etc/default/ntpd.conf
1. Restart ntpd(8) by invoking ```systemctl restart ntpd.service```
1. Verify ntpd(8) by invoking ```systemctl status ntpd.service```
1. Verify ntpd(8) is reading via shared memory/PPS
![ntpq](https://github.com/guycole/perky-janus/blob/main/grafix/ntpq.png)

### Relevant Links
1. [thread w/RCN](https://forum.beagleboard.org/t/beaglebone-black-gps-pps-and-chrony-for-time-sync/897/17)
1. [overlays on elinux.org](https://elinux.org/Beagleboard:BeagleBoneBlack_Debian#U-Boot_Overlays)
1. [pps on kernel.org](https://www.kernel.org/doc/html/latest/driver-api/pps.html)
1. [gpsd time service howto](https://gpsd.gitlab.io/gpsd/gpsd-time-service-howto.html)
1. [beaglebone-gps-clock](https://github.com/jrockway/beaglebone-gps-clock)
1. [Dan Drown BBB as NTP/GPS](https://blog.dan.drown.org/beaglebone-black-ntpgps-server/)
1. [toptechboy python](https://toptechboy.com/beaglebone-black-gps-tracker-lesson-3-parsing-the-nmea-sentences-in-python/)
1. [small golden sceptre](https://mythopoeic.org/beaglebone-green-time-server/)
1. [LinuxPPS Wiki](http://linuxpps.org/doku.php)

