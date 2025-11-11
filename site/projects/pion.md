**[Pion: Custom Arduino Design                                    08/2019 - 03/2022
--------------------------------------------------------------------------------]**

When I was working on an Arduino project several years back, one of the biggest
issues was the footprint of the device I had created. Because I was using an
Arduino Uno, the only way to supply power portably was to connect a pack of AA
batteries to the Uno's power jack, which unfortunately resulted in the device
being both large and heavy. The alternative was to use a 3.3V board with a LiPo
battery, but at the time the only one I had was an ESP32 based board, which I
had trouble programming. Frustrated by this, I decided to make my own. After
numerous failed attempts, I finally succeeded, and this was the result. I chose
to call it Pion, after the subatomic particle of the same name, to highlight its
small size.

#[
!![pion from top](/img/pion/pion_top.jpg)[39]  !![pion from bottom](/img/pion/pion_bottom.jpg)[39]
]#
*[The top and bottom of Pion. Everything here was hand-soldered]* 💀*[
--------------------------------------------------------------------------------]*

![pion by quarter](/img/pion/pion_quarter.jpg)(/img/pion/pion_quarter.jpg)
*[Pion is about the size of a US quarter.
--------------------------------------------------------------------------------]*

![size comparison](/img/pion/pion_comparison.jpg)(/img/pion/pion_comparison.jpg)
*[Pion next to a bunch of other popular Arduinos. It's the smallest of them all!
--------------------------------------------------------------------------------]*

Being based off the AtMega328, Pion is functionally equivalent to an Uno, Nano,
or Pro Mini, while being smaller, and in my opinion more versatile. The battery
connector allows you to power it with a rechargeable LiPo, and the female header
pins make it easy to connect short breadboard wires or design shields (expansion
boards that plug into the top) which are traditionally only available for the
Uno. To illustrate this, I also made this USB-C shield for Pion since it
normally doesn't have a USB port for programming:

#[
!![usb shield](/img/pion/pion_shield.jpg)[39]  !![usb shield](/img/pion/pion_shield_stack.jpg)[39]
]#
*[The USB shield I made for Pion, in action. Soldering was a pain.
--------------------------------------------------------------------------------]*

Making this was a tedious yet rewarding process; I learned a lot about how
Arduinos work, gained experience with designing custom PCBs, and refined my
soldering skills (those 0402 capacitors were the worst). And in the process, I
made a tiny, cool-looking board to show off and use in projects.

By the time I finally got this working, I had moved on from the project that
inspired its creation. But having made it, I had to use it:
#![video/mp4](/img/pion/pion_car.mp4)
#![video/mp4](/img/pion/pion_flappy.mp4)

[<<< Go back](/projects)
