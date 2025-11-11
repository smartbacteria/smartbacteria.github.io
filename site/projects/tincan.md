**[Tin Can Linux: A Custom Linux Distribution                     07/2024 - present
--------------------------------------------------------------------------------]**

This summer I began working on a custom Linux package manager, and soon after
began to design a custom distribution using this package manager. This is a
place for me to write about how I got here and write about the process. For
specifics about the distribution and installation instructions, see the official
website at ?[https://tincan-linux.github.io].


=== A Brief Overview
====================

Tin Can Linux is an independent hobby distribution made with the goal of being
compact, understandable, hackable, and easy to maintain. It aims to achieve
this by using unconventional packages that reduce the footprint of the
distribution, eliminating unnecessary dependencies, and making some
modifications to the standard file system layout. It also uses a custom
package manager to handle installation and removal of software.

At the moment, I have added enough packages the to repositories to start a
graphical session with a terminal and web browser (NetSurf). Here's some
screenshots of Tin Can in action:

![sysfetch](/img/tincan-1.png)(/img/tincan-1.png)

![browser](/img/tincan-2.png)(/img/tincan-2.png)

![editor](/img/tincan-3.png)(/img/tincan-3.png)


I am continuing to work on adding a few more packages, completing the transition
from Xorg to Wayland, and shrinking the footprint of the distro.

Tin Can is still in its initial stages; however, at the moment it is reasonably
stable, and I am able to use it for simpler tasks like programming and web
browsing. It also has a small base of around 10 users.


=== How and Why?
================

I have wanted to make my own Linux distribution ever since I discovered Linux
From Scratch (LFS) a few years back. The first time I tried it, I was eventually
able to work through the build process, but much to my dismay it wouldn't boot.
Since then I've revisited the idea on and off, with further attempts to make my
LFS boot, exploring Kiss Linux, and taking on the simpler task of making a
minimal distro for the Raspberry Pi Zero based on Cross-LFS Embedded. Though the
process has been slow, taking on these little projects and spending more time
actively using Linux has helped me gain a better understanding of how it works.
So, when I decided to try my hand at making a custom distribution again, it
actually panned out, and Tin Can is the result.

I primarily started doing this just for the fun of it. This project has been a
nice thing to have going on the side, to occasionally fall back on when I'm
bored, stressed, or just need a quick distraction. But aside from that, one of
my motivations for making this was to try making a distro that was as compact
as possible and required the fewest packages while still being reasonably
usable. I've always been curious about the ways that I could push an operating
system to the limits specifically in terms of the footprint, and the flexibility
and control that Linux offers has made it a perfect testing ground to satiate
this curiosity.

And finally, I've been inspired by various other really cool and unique projects
from the community, including:

  - Kiss Linux (?[https://kisslinux.github.io])
  - StaLi (?[https://sta.li])
  - Glaucus (?[https://glaucuslinux.org])
  - Oasis Linux (?[https://github.com/oasislinux])
  - Linux From Scratch (?[https://www.linuxfromscratch.org])

to name a few of the most direct influences.


=== Future Direction
====================

For now, I plan to finish transitioning the distribution from Xorg to Wayland as
the choice of display server protocol and work on further simplifying the file
system layout. I also have plans to make some changes to which packages are
included (change OpenSSL to BearSSL, bison to byacc, flex to reflex, etc) and
try removing more dependencies.

[<<< Go back](/projects)