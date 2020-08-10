---
title: "FTDI problem: PID 0000"
header:
    teaser: /assets/ftdi_problem_pid_0000/arduino-ftdi-vignette1-e1428432287781-1038x576.jpg
tags: [ "FTDI", "Hardware", "ft232r", "eeprom", "USB" ]
---

On 29 September 2014, FTDI released an updated version of their USB-to-Serial driver, It was reported by some users that the updated can brick some fake FTDIs chips, by changing their ID to 0000\. After some time, the windows update the version of the drive, now this problem is happening a lot in the last days.

# Solving the problem

1.  Use a linux machine (Ubuntu, mint and etc)
2.  Download [ft232r_prog-1.24.tar](http://rtr.ca/ft232r/ft232r_prog-1.24.tar.gz)
3.  Now install make, gcc and libftdi-dev ($ sudo apt-get install make gcc libftdi-dev)
4.  Unzip and compile the ft232r_prog ($ tar -xvf ft232r_prog-1.24.tar.gz, $ make)
5.  Connect the FTDI on your computer.
6.  Now change the ID (sudo ./ft232r_prog --old-pid 0x0000 --new-pid 0x6001)
7.  Disconnect and connect again the FTDI and run the lsusb command, now you will see that de id will be 6001.
8.  Have fun !

###### TLDR

```sh
$ wget http://rtr.ca/ft232r/ft232r_prog-1.24.tar.gz
$ tar -xvf ft232r_prog-1.24.tar.gz
$ cd ft232r*
$ sudo apt-get install make gcc libftdi-dev
$ make
$ sudo ./ft232r_prog --old-pid 0x0000 --new-pid 0x6001
```