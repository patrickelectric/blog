---
title: "Starting with Intel® Edison Platform"
header:
    teaser: /assets/starting_with_intel_edison_platform/Intel_Edison_with_stamp_nr.jpg
tags: [ "Intel", "Hardware", "Embedded", "Linux", "Tutorial", "Edison" ]
---

The Edison Platform is a little computer with all necessary system and hardware to provide a good experiment, we have: wi-fi, bluetooth, serial, io pins and alot more.


As we see the Edson board is an x86-32 architecture, 4 gigabytes to store our files, 1 gigabyte of ram to execute our applications, 2 processing cores to allow the development of parallel programs. this is a development platform with higher performance compared to Respaberry Pi  and 1/40 of its volume.

# Starting

You will need two  USBs ([micro B](http://www.usbfirewire.com/usb_cables_a_to_micro-b_non-angled_67.html)), one is for serial communication and the other is for power, ethernet, arduino upload and storage device.

Connecting the two cables in your computer, you can use [Putty](http://portableapps.com/apps/internet/putty_portable) if you are using windows, or you can use screenin your linux computer.

<pre>$ screen /dev/ttyUSB0 115200</pre>

After connect, type two times the enter key and now we can access the terminal from the [Yocto Linux](https://www.yoctoproject.org/). The login can be root/root or edison/edison.

# Configuration

Now, we can configure Edison with the command `$ configure_edison --setup `, we can configure almost everything including the Wi-Fi. The board have already an ssh-server enabled by default.

<pre>$ configure_edison --setup</pre>

# Installing programs

The Yocto don't have a package manager with the usual programs, so we need to download the *.deb file of the x86-32 architecture.

An example, to install nemo (the text editor):

<pre>$ wget http://www.nano-editor.org/dist/v2.2/nano-2.2.0.tar.gz
$ tar -xvf nano-2.2.0.tar.gz
$ cd  nano*
$ ./configure
$ make
$ make install</pre>

# Installing programs with opkg

The Intel have a [repository](https://software.intel.com/en-us/iot) from the Yocto projecto to the Edison, we can upgrade or system repository to use the Intel IoT devkit.

<pre>nano /etc/opkg/base-feeds.conf</pre>

Inside this file, we need to write the local of the Intel repositories.

<pre>src all http://iotdk.intel.com/repos/1.1/iotdk/all
src x86 http://iotdk.intel.com/repos/1.1/iotdk/x86
src i586    http://iotdk.intel.com/repos/1.1/iotdk/i586</pre>

After that, we can upgrade or package manager database.

<pre>opkg update</pre>

And install something util.

<pre>opkg install git</pre>