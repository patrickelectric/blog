---
title: "[GSoC] KDev-Embedded, OpenOCD and avrdude"
tags: [ "KDE", "Hardware", "Arduino", "ARM", "avrdude", "OpenOCD" ]
---

KDev-Embedded  now have OpenOCD integration and a new interface to use avrdude in launcher.

With [Arduino-Makefile](https://github.com/sudar/Arduino-Makefile), it's possible to use a makefile to perform compilation of Arduino projects. In the video one the the examples are used to shows how it is possible to use the new avrdude launcher to execute the upload process.

<video width="640" height="360" controls>
  	<source src="/assets/kdev_embedded_openocd/arduino1.mp4" type="video/mp4">
</video>

In the avrdude new interface was added more KComboBox to create a more generic and configurable interface helping advanced users.

The OpenOCD support can upload the binary to an embedded plataform and launch the OpenOCD server to perform upload and debugging with GDB. The graphic interface still in development for further improvements.

<video width="640" height="360" controls>
  	<source src="/assets/kdev_embedded_openocd/lm4f2321.mp4" type="video/mp4">
</video>

In the next steps, we are aiming the OpenOCD interface to be more friendly with basic and advanced users, and a new integration with DFU or other tool for embedded systems.

Best regards,