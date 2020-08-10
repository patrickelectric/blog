---
title: "[GSoC] KDev-Embedded, workflow integration"
header:
    teaser: /assets/kdev_embedded_workflow/arduinowindow065.png
tags: [ "KDE", "Hardware", "Arduino", "ARM", "avrdude", "OpenOCD" ]
---

After some work in the plugin development, now the project have a strong focus in a better integration with KDevelop workflow. Until now the Board Configuration window have some simple features to perform the upload process for beginner users, it's called by the _**embedded**_ submenu in the KDevelop toolbar.

<figcaption class="wp-caption-text">Welcome message</figcaption>

![arduinowindow063](/assets/kdev_embedded_workflow/arduinowindow063.png)

<figcaption class="wp-caption-text">Error message</figcaption>

![arduinowindow064](/assets/kdev_embedded_workflow/arduinowindow064.png)

<figcaption class="wp-caption-text">Success message</figcaption>

![arduinowindow065](/assets/kdev_embedded_workflow/arduinowindow065.png)


The problem is that the Board Configuration window don't follow the integrated workflow of KDevelop, and that's what are we doing right now, turning the **KDev-Embedded** an integrated plugin helping programmers that already know how to use the software and how to perform what they want. That's the idea behind the **Embedded Launcher**.

<figcaption class="wp-caption-text">KDev-Embedded Embedded Launcher</figcaption>

![launch_config70](/assets/kdev_embedded_workflow/launch_config70.png)

The Embedded Launcher try to help beginners and advanced users with presets and a Board menu, until now the preset ComboBox is disabled until we finish the ARM support and the Board menu configure some others features like MCU, Interface baud rate and Arguments, but the user is free to perform modifications in this fields thanks to KComboBox. After the launcher configuration finished the user can save and execute it to perform  the upload process that shows in KDevelop output.

<figcaption class="wp-caption-text">KDevelop output showing the programmer feedback</figcaption>

![selection074](/assets/kdev_embedded_workflow/Selection_074.png)

The plugin still in development, supporting  Arduino board and being tested only on some boards like Arduino Nano and Mini. In the next week we'll start the ARM support.