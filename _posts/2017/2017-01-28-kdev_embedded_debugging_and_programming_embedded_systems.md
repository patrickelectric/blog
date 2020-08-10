---
title: "[GSoC] KDev-Embedded, Debugging and programming embedded systems"
tags: [ "KDE", "Hardware", "Arduino", "ARM", "avrdude", "OpenOCD" ]
---

The actual embedded system word depends on closed-source IDEs and libraries, with high monetary value and deprecated functionalities. Programmers that would like to use ARM based boards without paying for an IDE will have problems setting up such development ambient and synchronized toolkits.

The main idea of this project is to provide a plugin integrated with KDevelop to help the debugging and programming process of embedded systems like AVR, ARM and x86 based boards.

Since 2011, starting with my electronic engineering graduation the contact with embedded system begun with PIC and ATMEL uC. In 2012 I was accepted in the ROBOTA (competition D&R team) and the ProVANT (Project and Development of UAV in the tilt-rotor configuration), both project working with hardware and software integration.

The development on ProVANT project showed how difficult can be to program an embedded system without using closed source software, It's hard to find a descent IDE to develop in such area, ever harder to do it with open-source programs and tools.

But after some time I discovered [OpenOCD](http://openocd.org/) and _arm-none-eabi_ to do my job. But it was such a pain first time to understand and use everything together but after some time everything start to progress.

Now I am at Intel doing my internship to finish my electronic engineering graduation and working on KDevelop for GSOC 2016 project to turn KDevelop into a native system to debug and program embedded systems.

Until now the plugin developed can download and install the Arduino toolkit. This part was strongly based on [ArduIDE](https://github.com/mupuf/arduide) project for which I have contributed by updating the Arduino supported version and some corrections of the code, with a great help of Mupuf. Also a window in development to configure that board and interface to program.

The GSOC submission can be found [here](http://patrickjp.com/wp-content/uploads/2016/04/gsoc.pdf) for download.

TLDR: The focus of the project is a versatile plugin that can be used for both first travel and experienced programmers in the word of embedded systems !

I'll try to update the blog as the project progress.
