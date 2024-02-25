# Here's the guide

Welcome, traveller! I suppose you're here to install Archlinux, don't wait then, let's go!

This guide will cover and explain more aspects of the installation than what you can see on the wiki [here](wiki.archlinux.org/title/Installation_guide)

![Archlinux finished install](./screenshots/archlinux.png)

# First commands :

If you haven't a QWERTY keyboards, you may want to change the layout :

`localectl list-keymaps` to list the keymaps (obviously)

and

`loadkeys us-acentos` < Here I load the US International layout, but choose what you want.

**For HiDPI monitors :**

`setfont ter-132b`

> [!NOTE]
> BIOS technology is outdated, I will consider you are using UEFI.

**Connect to the internet** 

This step is only for wifi users.

There are two network managers available in the installation medium, I will use `network-manager`.

`nmcli device` to list your network interfaces.
`nmcli device wifi list` to list the available access points.
`nmcli device wifi connect [SSID] password [password]` (replace the items inside brackets)

You should be connected, try to send a ping to check your connection :

`ping 1.1.1.1`

If there's a response, it works! Next step.

It's time to set up the system clock :

`timedatectl set-ntp true`

Fine, let's go ahead.

# Partitioning 
