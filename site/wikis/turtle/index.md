|-----------------------|
| TURTLE                |           ()---()
|-----------------------|       ,--/ ()()()\\
| A tiny window manager |       |: |()()()()|>
| written in Rust.      |       `--\\ ()()()/
|-----------------------|           ()---()
=~=~=~=~=~=~=~=~=~=~=~=~=

This is the webpage for turtle. For general information about building and
installing, see [the github repo](https://github.com/avs-origami/turtle).

=== CONFIGURATION
=================

Turtle will look for a configuration file at ~/.config/turtle/config.ron. An
example config file is provided in the repo. Turtle also supports running a
startup script at ~/.config/turtle/autostart.

=== INFO FILE
=============

Turtle provides an info file at ~/.config/turtle/info.txt. Currently this
outputs a single line with the format 'focused:[id]' which gives the id of
the currently focused window. This may be useful for use in shell scripts or
statusbars.

=== KEYBINDS
============

The default keybinds for turtle are listed below:
  - Super + Shift + Q: quit turtle
  - Super + W: kill focused window
  - Super + F: make a window fullscreen (ignores all gaps)
  - Super + T: make a window large (respects gaps)
  - Super + S: make a window small
  - Super + [: cycle windows left
  - Super + ]: cycle windows right
  - Super + Tab: switch to the last focused window
  - Super + Enter: open a terminal (st)
  - Super + Space: open rofi in run mode

Additionally, the following mousebinds are set:
  - Super + LMB: move the selected window
  - Super + RMB: resize the selected window

All keybinds and mousebinds can be set in the config file.
