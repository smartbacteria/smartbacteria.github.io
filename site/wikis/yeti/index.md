|-------------------------|     ,_____,
| YETI OS                 |     | . . |
|-------------------------|     | ._, |
| A bit-sized linux       |     |-----|
| distro for RPi zero     |    / ,   , \\
|-------------------------|    |_'   '_|
|=~=~=~=~=~=~=~=~=~=~=~=~=|     '--^--'

This is the webpage for Yeti OS. For general information on building and
installing, see [the github repo](https://github.com/smartbacteria/yeti).

=== INSTALLED PACKAGES
======================

Yeti OS comes with the following packages installed:
  - musl
  - binutils
  - gcc
  - busybox
  - linux
  - ypm
  - make
  - wpa_supplicant

None of the binaries are stripped during the build process. Stripping binaries
can reduce the size of the final system. Additionally, any binaries beginning
with `armv6zk-linux-musleabihf-` can be safely removed. These are binaries used
in cross-compilation and are not useful on the final system.

=== PACKAGE MAANGEMENT
======================

[The Yeti Package Manager (YPM)](/projects/ypm) is used for package management and is built into
the system by the build script.

Basic usage:
ypm -i [PACKAGE] to install [PACKAGE]
ypm -r [PACKAGE] to uninstall [PACKAGE]

See the YPM page and github repo for more detailed explanations.

=== FILESYSTEM
==============

The filesystem is based on the Static Linux filesystem
(https://sta.li/filesystem). The directories are as follows:

/bin - executables
/dev - devices
/etc - system config and packages
/home - user dirs
/include - headers
/lib - libraries
/opt - weird packages
/proc - proc files
/run - run files
/sbin -> bin
/share - share stuff
/sys - sys files
/usr -> /
/var - var stuff

This could change in future. Fewer directories = simpler system.

If new directories or files start popping up in / it's probably because they
were going to be installed to /usr. If you want you can make /usr a directory
instead of symlinking to / but this was done for simplicity.

=== RESOURCES
=============

Resources used throughout this project:
  - Linux From Scratch (https://www.linuxfromscratch.org)
  - Cross Linux From Scratch (https://clfs.org)
  - Mussel (https://github.com/firasuke/mussel)
  - Diy Linux Guide (https://github.com/AgentD/diy-linux-guide)
  - PiLFS (https://intestinate.com/pilfs)
  - Static Linux (https://sta.li)
