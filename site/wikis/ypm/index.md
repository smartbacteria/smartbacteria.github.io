|----------------------------|     ,_____,
| Yeti Package Manager (YPM) |     | . . |
|----------------------------|     | ._, |
| A linux package manager    |     |-----|
| written in Rust.           |    / ,   , \\
|----------------------------|    |_'   '_|
=~=~=~=~=~=~=~=~=~=~=~=~=~=~=~     '--^--'

This is the webpage for YPM. For general information on building and
installing, see [the github repo](https://github.com/avs-origami/ypm).

=== USAGE
=========

ypm [OPTIONS] [ARGUMENTS]

Available options:
  -c => Clear the package cache
  -d => Download the sources for a package
  -e => Exit immediately on fail
  -g => Generate a package template using the package generator utility
  -h => Show this help
  -i => Install a package
  -o => Set offline mode (may be removed in future)
  -r => Remove a package
  -v => Get version information

You could also view the man page by running `man ypm`.

=== PACKAGE GENERATOR
=====================

The package generator is a utility that can be used to easily add your own
packages to your local repositories.

Usage:
ypm -g [name] [ver] [repo] [src] [no_src] [has_install] [deps] [mkdeps] [extras]

name: package name
ver: package version
repo: core or extra
src: URL of source tarball
no_src: set true if there is no source tarball
has_install: set true if this package should have an install script
deps: comma separated list of dependencies, use "" for none
mkdeps: comma separated list of make dependencies, use "" for none
extras: URLs of additional sources (comma separated), use "" for none

A text editor will open to modify the build and install scripts. If $EDITOR is
set that program will be used, otherwise vim will be used.

In future there may be an interactive mode, for now all arguments have to be
supplied at the command line.

=== AVAILABLE PACKAGES
======================

The following packages are already in the repo for ypm:

acl, attr, autoconf, automake, bash, bc, binutils, bison, busybox, bzip2, check,
coreutils, dbus, dejagnu, diffutils, e2fsprogs, expat, expect, file, findutils,
flex, gawk, gcc, gdbm, gettext, glibc, gmp, gperf, grep, groff, grub, gzip,
iana-etc, inetutils, intltool, iproute, jinja, kbd, kernel, kmod, less, libcap,
libelf, libffi, libpipeline, libtasn, libtool, m4, make, make-ca, man-db,
man-pages, markupsafe, meson, mpc, mpfr, ncurses, ninja, openssl, p11-kit,
patch, perl, pkg-config, procps-ng, psmisc, python, readline, rust, sed, shadow,
sudo, tar, tcl, texinfo, util-linux, vim, wget, wheel, wpa_supplicant,
xml-parser, xz, ypm, zlib, zstd
