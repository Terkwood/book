# Binding alacritty to Gnome Shortcut Key

I want to hit a single key and focus on `alacritty` while
using GNOME.

## Start alacritty with a title when logging into GNOME

This is much harder than it should be.

See [this article on creating a startup command in gnome](https://forum.manjaro.org/t/what-is-the-easiest-way-to-create-a-startup-command-in-gnome/30506/3)

Write a script:

```sh
#!/bin/bash

alacritty --title=alacrittzam
```

Write `~/.config/autostart/alacritty.desktop`:

```text
[Desktop Entry]
Name=Alacritty
Exec=/home/itmefriend/bin/alacritty_named.sh
Terminal=true
Type=Application
StartupNotify=false

```

Now `alacritty` will start whenever you log in to GNOME.

## Bind key to focus

Install `wmctrl` so that you can raise the titled `alacritty` window.

```sh
apt install wmctrl
```

Create a script to raise `alacritty`:

```sh
#!/bin/bash
wmctrl -a alacrittzam
```

Finally, in gnome settings, you will need to add a keyboard shortcut which
runs your script.
