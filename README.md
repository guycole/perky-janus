# perky-janus
BeagleBone Black as a NTP server using GPS

## Introduction
This project is to share my experience using a [BeagleBone Black](https://beagleboard.org/black) to obtain time from a GPS and then sharing time with other computers on a local network.  There are many stale posts about this topic, so here is a fresh take to throw on the pile.  I am writing this is May, 2022 for the BeagleBone Black (Revision C) using [AM3358 Debian 10.3 2020-04-06 4GB SD IoT](https://debian.beagleboard.org/images/bone-debian-10.3-iot-armhf-2020-04-06-4gb.img.xz)

My context is a network of single board computers in a mobile application, the time need only be "reasonably" accurate (i.e. within a second) and it must run unattended for extended perios.  Little computers have terrible time drift, so an external source was a priority.  The answer could have been a RTC, but I also needed location (because, mobile) which made a GPS mandatory.  Why not get time from the GPS?

I have since learned that USB GPS receivers would not readily interface with ntpd(8) or chronyd(8), because the kernel driver needs PPS.  
