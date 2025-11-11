**[Installing Kiss Linux                                                 01/14/2024
--------------------------------------------------------------------------------]**

I recently tried installing Kiss Linux, and as the guide hasn't been updated in
a bit, I've run into some issues along the way. I thought that I would make a
post with some annotations to the original Kiss Linux installation guide that
talks a bit about my experience installing the distribution from the community
repositories.

Section numbers in this guide correspond to the sections at
https://kisslinux.github.io/install.


=== [001] Installing
====================

This assumes that you have partitioned your hard disk and mounted it to /mnt.
For a guide on partitioning see
https://nixos.org/manual/nixos/stable/#sec-installation-manual-partitioning

Download and unpack the tarball as usual:

   ver=23.04.30
   url=https://codeberg.org/kiss-community/repo/releases/download/$ver
   file=kiss-chroot-$ver.tar.xz
   curl -fLO $url/$file
   cd /mnt \&\& tar xf $OLDPWD/$file
   /mnt/bin/kiss-chroot /mnt


=== [007] Repositories
======================

Clone the repos (It doesn't matter where, I use /) and set KISS_PATH:

   git clone https://codeberg.org/kiss-community/repo.git
   echo "export KISS_PATH=/repo/core:/repo/extra:/repo/wayland" >> /etc/profile
   source /etc/profile


Enable signature verification:

   cd /repo
   git config gpg.ssh.allowedSignersFile .allowed_signers
   git config merge.verifySignatures true


=== [014] Rebuild the system
============================

Set the compiler flags (it may be helpful to add these lines to /etc/profile):

   export CFLAGS="-O3 -pipe -march=native"
   export CXXFLAGS="$CFLAGS"
   export MAKEFLAGS="-j$(nproc)"


Update the packages:

   kiss update
   cd /var/db/kiss/installed \&\& kiss b *


=== [018] Kernel
================

Download and unpack the kernel sources to somewhere you remember, such as ~:

   curl -fLO https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.7.tar.xz
   tar xvf linux-6.7.tar.xz
   cd linux-6.7


Make a default kernel config, then start menuconfig to customize the options:

   kiss b ncurses pkgconf
   make defconfig
   make menuconfig


The kernel configuration can be hard to get right, and missing something here
will probably be the reason that Kiss doesn't boot. I personally struggled the
most with this step of the process. Here are some tips:

Running `make localyesconfig` from an Arch Linux live ISO can help to enable
some options that you might need; however, you will likely need to configure the
kernel yourself beyond this.

Running `lsmod` from the live environment can help to tell what kernel modules
are needed, and `lspci` can tell you what hardware your computer uses. Pay
attention to any output in either of these that references a graphics card or
driver since missing graphics drivers can cause your system to not boot.

Also, `dmesg` can help to determine what firmware may be needed, and can further
reveal kernel options that need to be enabled.

After configuring through menuconfig, run
`grep 'FB' .config \&\& grep 'FRAMEBUFFER' .config` in order to ensure that you
have all the framebuffer drivers enabled. These are a bunch of framebuffer
drivers that I enabled. You may need some combination of these - play around and
see what works for you:

    CONFIG_SYSFB
    CONFIG_SYSFB_SIMPLEFB
    CONFIG_DRM_FBDEV_EMULATION
    CONFIG_FB
    CONFIG_FB_VESA
    CONFIG_FB_VGA16
    CONFIG_FB_EFI
    CONFIG_FB_CORE
    CONFIG_FB_NOTIFY
    CONFIG_FB_DEVICE
    CONFIG_FRAMEBUFFER_CONSOLE


Additionally, [this Gentoo Wiki page](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Kernel/en#Kernel_configuration_and_compilation) can be a useful resource for kernel
configuration. You may be able to find a dedicated page for installing Gentoo
on your specific computer, like [this one I found for the Framework Laptop](https://wiki.gentoo.org/wiki/Framework_Laptop_13).

You might also need some additional firmware not included in the kernel, such as
Intel Ucode drivers 

Finally, if you're still having issues, ask on the IRC (#kisslinux), check
[the logs](https://libera.irclog.whitequark.org/kisslinux/), or ask on Reddit at [r/kisslinux](https://reddit.com/r/kisslinux).

With the kernel configured, continue to building it:

   kiss b libelf
   patch -p1 < /usr/share/doc/kiss/wiki/kernel/no-perl.patch
   sed '/<stdlib.h>/a #include <linux/stddef.h>' \\
   tools/objtool/arch/x86/decode.c > _
   mv -f _ tools/objtool/arch/x86/decode.c
   sed -i 's/=m/=y/g' .config  # Change modules to baked-in
   make
   make install
   mkdir -pv /boot/EFI/kiss
   cp arch/x86/boot/bzImage /boot/EFI/kiss/bzImage.efi
   mv /boot/vmlinuz /boot/vmlinuz-6.7
   mv /boot/System.map /boot/System.map-6.7


Something I found that you probably want to do is change all modules to baked-in
so that you don't have to worry about loading all the correct modules at boot.
Normally this might be handled by the initramfs, but Kiss doesn't use one so
I have found it's just easier without kernel modules. The command to do this is
part of the list of commands shown above, but if you know what you're doing and
want kernel modules, it's not strictly required.

Especially if you expect to have problems in the kernel configuration, it may be
worthwile to (semi) automate the process of building the kernel with a shell
script. Interestingly enough, the Kiss package manager supports creating
extensions by naming a binary kiss-[ext], and I found it nice to create the
kernel update script as a kiss extension. Add this to /bin/kiss-kernel:

   #!/bin/sh -e
   # Rebuild the kernel
  
   log() {
       printf '\\033[32m->\\033[m %s.\\n' "$*"
   }
  
   die() {
       log "$*" >\&2
       exit 1
   }
  
   [ "$(id -u)" = 0 ] || die Script needs to be run as root
   log Building and installing kernel version $1
   cd $HOME/linux-$1
   make menuconfig
   sed -i 's/=m/=y/g' .config
   make
   log Installing kernel modules
   make INSTALL_MOD_STRIP=1 modules_install
   log Installing kernel to /boot
   make install
   cp arch/x86/boot/bzImage /boot/EFI/kiss/bzImage.efi
   mv /boot/vmlinuz-$1 /boot/old_vmlinuz
   mv /boot/System.map-$1 /boot/old_System.map
   mv /boot/vmlinuz /boot/vmlinuz-$1
   mv /boot/System.map /boot/System.map-$1
   log Successfully installed kernel to /boot


Then make it executable:

   chmod +x /bin/kiss-kernel


Now you can update the kernel by running `kiss kernel 6.7` as root.

Disclaimer: I created this script because I was constantly changing the kernel
config and rebuilding it during the install process. If you've figured out a
kernel config that works well and don't plan to change it, you should consider
packaging the kernel instead (see [https://kisslinux.github.io/package-system]).


=== [029] Making the system bootable \& finishing touches
========================================================

Install the init system:

   kiss b baseinit

Create /etc/fstab (apparently, this isn't strictly required):

   cat > /etc/fstab << "EOF"
   # Begin /etc/fstab

   # file system  mount-point    type     options             dump  fsck
   #                                                                order

   LABEL=boot     /boot          vfat     defaults            0     2
   LABEL=kiss     /              ext4     defaults            0     1
   LABEL=swap     swap           swap     pri=1               0     0
   proc           /proc          proc     nosuid,noexec,nodev 0     0
   sysfs          /sys           sysfs    nosuid,noexec,nodev 0     0
   devpts         /dev/pts       devpts   gid=5,mode=620      0     0
   tmpfs          /run           tmpfs    defaults            0     0
   devtmpfs       /dev           devtmpfs mode=0755,nosuid    0     0
   tmpfs          /dev/shm       tmpfs    nosuid,nodev        0     0
   cgroup2        /sys/fs/cgroup cgroup2  nosuid,noexec,nodev 0     0

   # End /etc/fstab
   EOF


Install GRUB if using BIOS:

   unset CFLAGS CXXFLAGS   # Grub doesn't compile with flags like -march=native
   kiss b grub
   grub-install --target=i386-pc /dev/sdX
   grub-mkconfig -o /boot/grub/grub.cfg

If on UEFI, use EFI Stub instead (make sure support is enabled in the kernel):

   kiss b efivar efibootmgr
   efibootmgr --disk /dev/sdX --part Y --create \\
     --label "Kiss EFI Stub" --loader '/EFI/kiss/bzImage/efi' \\
     --unicode 'root=PARTUUID=XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX rw'


Re-set the compiler flags, and install some extra software:

   source /etc/profile
   kiss b e2fsprogs dosfstools eiwd dhcpcd


Set a root password and add a normal user:

   passwd root
   adduser <username>
   passwd <password>


For further information, these sites may be useful:
https://kisslinux.github.io
https://kisscommunity.org
https://github.com/kiss-community/awesome-kiss

[<<< Go back](/blog)