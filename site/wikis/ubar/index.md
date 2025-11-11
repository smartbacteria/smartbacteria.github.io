|-----------------------------|
| UBAR: micro-sized statusbar |
|-----------------------------|

A micro-sized statusbar for linux.

- Incredibly minimal (only 103 lines with comments)
- Depends only on xlib and libxft
- Configured entirely in code

=== BUILDING
============

1. Edit `config.h`
2. Run `make`
3. Move `ubar` to your bin directory

=== SHOWING INFO
================

The information that is shown on the bar is determined by running two shell
scripts which can be set in `config.h`. Examples are provided in the repository
at `bar_left.sh` and `bar_right.sh`.