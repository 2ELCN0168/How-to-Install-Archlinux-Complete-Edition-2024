# Here's the guide

Welcome, traveller! I suppose you're here to install Arch Linux, don't wait then, let's go!

This guide will cover and explain more aspects of the installation than what you can see on the wiki [here](wiki.archlinux.org/title/Installation_guide)

![Archlinux finished install](./screenshots/archlinux.png)

# Before starting :

- Some computers won't work out of the box because of bugs left by the manufacturers ;
- Arch Linux comes with an installer, if you want to setup your system quickly or make tests, you'e better use the `archinstall` script. For a real installation, you should follow this guide ;
- This guide is for UEFI only.

# First commands :

If you haven't a QWERTY keyboards, you may want to change the layout :

- `localectl list-keymaps` to list the keymaps (obviously).

and

- `loadkeys us-acentos` < Here I load the US International layout, but choose what you want.

**For HiDPI monitors :**

- `setfont ter-132b`.

> [!NOTE]
> BIOS technology is outdated, I will consider you are using UEFI.

**Connect to the internet** 

This step is only for wifi users.

- Run the `iwctl` utility.
- If you want help, type `help`.
- `device list`.
- If the device is turned off, `device *device* set-property Powered on` then `adapter adapter set-property Powered on`.
- `station *device* scan`.
- `station *device* get-networks`.
- `station *device* connect *SSID*`.
- Then we can Ctrl+d to exit.

You should be connected, try to send a ping to check your connection :

- `ping -4c4 1.1.1.1`.

If there's a response, it works! Next step.

It's time to set up the system clock :

- `timedatectl set-ntp true`.
- `timedatectl status` to check everything is fine.
Fine, let's go ahead.

# Partitioning 

This part is quite subjective, there are a lot of ways to do it. I will covers some aspects here but you should define your needs first and make more researchs about what you're about to do.

- Do `lsblk` to identify your disk(s), if you have a SATA connected drive, it will be named *sdX* where "X" is the letter of the disk. If you have a nvme ssd, it will be named *nvmeXn1* where "X" is the ssd number.
- We will use `gdisk` to partition our disk(s).

### Classical way

- Do `gdisk /dev/sd*X*`.
