---
title: "The real tutorial for Galileo Gen 2 Board"
header:
    teaser: /assets/the_real_tutorial_for_galileo_gen_2_board/galileo-front-2x1-1038x576.jpg
tags: [ "Intel", "Hardware", "Embedded", "Linux", "Tutorial", "Galileo" ]
---

Some of the tutorials on the internet are outdated, and cause a lot of problems to start on the develop of the Galileo Gen 2 Board, to solve this, we will explain what you'll need to do to make the best setup possible.

1.  Test tour board.
    Make a simple test, take a FTDI cable.

    ![ftdi](/assets/the_real_tutorial_for_galileo_gen_2_board/ftdi5v.jpg)

    Put the black cable (the one that have an arrow on it) on the GND.

    ![Galileo Connector](/assets/the_real_tutorial_for_galileo_gen_2_board/ftdi-galileo.jpg)

    The board come with a jumper to select the logic level of the FTDI (5V or 3.3V).
    After that, we  can access the terminal of the inboard Linux.  Plug the FTDI in your computer.

    <pre>$ dmesg</pre>

    With this command you will see the name of the USB adapter.

    ![ftdi dmesg](/assets/the_real_tutorial_for_galileo_gen_2_board/dmesg-ftdi.png)

    As we can see, the name of the USB adapter is ttyUSB0.
    Now we can connect with the board and read the feedback of the boot.

    <pre>$ screen /dev/ttyUSB0 115200</pre>

    Turn on your board and wait, you will see the boot options and the log.

2.  Upgrade your firmware.
    Download the program to update the firmware of the board.
    Website :[Intel Links.](https://downloadcenter.intel.com/download/24748/Intel-Galileo-Firmware-and-Drivers-1-0-4)
    Windows software : [IntelGalileoFirmwareUpdater-1.0.4-Windows](/assets/the_real_tutorial_for_galileo_gen_2_board/IntelGalileoFirmwareUpdater-1.0.4-Windows.zip)
    OS Independent : [IntelGalileoFirmwareUpdater-1.0.4](/assets/the_real_tutorial_for_galileo_gen_2_board/IntelGalileoFirmwareUpdater-1.0.4.jar)
    The Windows 10 and the Linux OS already have the drivers for Galileo and Edison.</span></span>
3.  Download the last image to your SD card.
    The last image can be downloaded with [this link.
    ](https://software.intel.com/sites/landingpage/iotdk/board-boot-image.html)Format the SD card with fat32 (you can do this with gparted).
    Unzip the file and put the image inside of the SD card, you can do this with the dd command.

    <pre>$ dd if=image of=/dev/sdx bs=8</pre>

    If the board did not boot the SD card try another card.

4.  Update the repository.
    Use the command `$ opkg update ` to update your the repository, if the command did not return no nothing, you will need to set the repository by hand, like done [in the this pots](/starting_with_intel_edison_platform). But use the new repository !
    You can see the new one [here](http://iotdk.intel.com/repos/).