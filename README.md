# Here's the guide

Welcome, traveller! I suppose you're here to install Arch Linux, don't wait then, let's go!

This guide will cover and explain more aspects of the installation than what you can see on the wiki [here](wiki.archlinux.org/title/Installation_guide)

![Archlinux finished install](./screenshots/archlinux.png)

# Before starting:

- Some computers won't work out of the box because of bugs left by the manufacturers ;
- Arch Linux comes with an installer, if you want to setup your system quickly or make tests, you'e better use the `archinstall` script. For a real installation, you should follow this guide ;
- This guide is for UEFI only.

# First commands:

If you haven't a QWERTY keyboards, you may want to change the layout:

- `localectl list-keymaps` to list the keymaps (obviously).

and

- `loadkeys us-acentos` < Here I load the US International layout, but choose what you want.

**For HiDPI monitors:**

- `setfont ter-132b`.

> [!NOTE]
> BIOS technology is outdated, I will consider you are using UEFI.

**Connect to the internet** 

This step is only for wifi users.

- Run the `iwctl` utility.
- If you want help, type `help`.
- `device list`.
- If the device is turned off, `device [device] set-property Powered on` then `adapter adapter set-property Powered on`.
- `station [device] scan`.
- `station [device] get-networks`.
- `station [device] connect *SSID*`.
- Then we can Ctrl+d to exit.

You should be connected, try to send a ping to check your connection:

- `ping -4c4 1.1.1.1`.

If there's a response, it works! Next step.

It's time to set up the system clock:

- `timedatectl set-ntp true`.
- `timedatectl status` to check everything is fine.
Fine, let's go ahead.

# Partitioning 

This part is quite subjective, there are a lot of ways to do it. I will covers some aspects here but you should define your needs first and make more researchs about what you're about to do.

- Do `lsblk` to identify your disk(s), if you have a SATA connected drive, it will be named *sdX* where "X" is the letter of the disk. If you have a nvme ssd, it will be named *nvmeXn1* where "X" is the ssd number.
- We will use `gdisk` to partition our disk(s).

- Do `gdisk /dev/sdX`.
- `o` to create a new empty GPT table.

### Classical way

We need at least two partitions. A boot partition and the system partition.
This is the very minimal required for your system to work.

If you don't want to use the hibernate function or if you have more than 16 GiB of RAM, you can skip the swap partition. I don't recommend it however.

Minimal partition size:
- `/` should be at least 23-32 GiB.
- `/boot` should be at least 1 GiB.

Now, you should know that the best practice is to have more than two partitions. 

RedHat recommends to have those partitions:

| Directory | Minimum size |
|-----------|--------------|
|    `/`    |     10 GiB   |
|  `/usr`   |     5 GiB    |
|  `/tmp`   |     1 GiB    |
|  `/var`   |     3 GiB    |
|  `/home`  |     3 GiB    |
|  `/boot`  |     1 GiB    |

- `n` to create a new partition.
    - From here, you have to fill the interactive fields.
    - Always leave the first sector by default and write "+[XX]G" for the partition size.

We will create the partitions in this order:

- `/boot`.
- (`swap`).
- `/`.
- etc.

The Hex codes are:
- `ef00` for EFI system partition (this should be for the 1st partition, `/boot`).
- `8200` for a swap partition.
- `8300` for a standard partition.

Once you have your partitions, you can type `c` to change their names.

- Type `p` to verify and `w` to write changes to the disk.

**Format the partitions:**

- `mkfs.fat -F 32 /dev/sdX` for the boot partition.
- `mkswap /dev/sdX` for the swap partition.
- `mkfs.ext4 /dev/sdX` for the other partitions.

- `swapon /dev/sdX` to activate the swap partition.
- `mount /dev/sdX /mnt` for the root partition `/`.
- `mount --mkdir /dev/sdX /mnt/boot` for the boot partition.
- `mount --mkdir /dev/sdX /mnt/[name of the partition]` for `/usr`, `/var`, `tmp`, etc.

- Then, do `lsblk` again to verify if everything is correct.

### Installation

- Open the file `/etc/pacman.conf` and uncomment the lines `#ParallelDownloads = 5` and `#Color`.
- Do `pacman-key --refresh-keys` to ensure the keyring is up to date.

Then, we will now install the base system:

- `pacstrap -i /mnt base{,-devel} dkms linux{{,-lts}{,-headers},firmware}`.

Here is the command I recommend to use, but note that it's my personnal preferences, select the packages you want instead of copying this line blindly:

- ```pacstrap -i /mnt base{,-devel} dkms linux{{,-lts}{,-headers},-firmware} git man-{db,pages} vim networkmanager openssh polkit zsh{,-{autosuggestions,completions,history-substring-search,syntax-highlighting}} tmux tldr texinfo```.

- Create the fstab file with `genfstab -U /mnt >> /mnt/etc/fstab`.

### System configuration

Now, it's time to do `arch-chroot /mnt` to go into your fresh installed system!

If you forgot to install packages, you can do `pacman -S [package name]` to install them.

To get the right date and hour according to your location, you have to do this command, make sure to replace "Europe" and "Paris" if you need to:

- `ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtime`.

- Then, do `hwclock --systhoc` to update the hardware clock.

- Open the file `/etc/locale.gen` and uncomment `en\_US.UTF-8` and any other locales you want to use.
- Do `locale-gen` to generate the uncommented locales.

- Do `echo LANG=[LOCALE] > /etc/locale.conf` **[LOCALE]** being your preferred locale from the ones you just generated.
- Do `echo KEYMAP=[KEYMAP] > /etc/vconsole.conf` **[KEYMAP]** being the name of the keymap you set up with `loadkeys` (if you did it).
- Do `echo FONT=lat0-16 >> /etc/vconsole.conf`. You can find all fonts available in `/usr/share/kbd/consolefonts`.

- Do `echo [HOSTNAME] > /etc/hostname` **[HOSTNAME]** being the name you want your system to have.
- The hostname must be compatible with the following regex expression: `^(:?[0-9a-zA-Z][0-9a-zA-Z-]{0,61}[0-9a-zA-Z]|[0-9a-zA-Z]{1,63})$`.

- Edit the file `/etc/hosts` to contain the following:

```bash
# Static table lookup for hostnames.
# See hosts(5) for details.

127.0.0.1 localhost.localdomain localhost localhost-ipv4
::1       localhost.localdomain localhost localhost-ipv6
127.0.0.1 HOSTNAME.localdomain  HOSTNAME  HOSTNAME-ipv4
::1       HOSTNAME.localdomain  HOSTNAME  HOSTNAME-ipv6
```
- **"HOSTNAME"** being the hostname you chose earlier.

- Enable some services with `systemctl enable`:
    - `sshd` if you want to connect to your machine by ssh ;
    - `NetworkManager` if you want some internet in your machine ;
    - `systemd-resolved` for the network name resolver service by systemd ;
    - `fstrim.timer` if you have a SSD.

- Edit the file `/etc/mkinitcpio.conf` and change `udev` to `systemd` in the `HOOKS` list.
- Update the initramfs by doing `mkinitcpio -P`.

- Do `passwd` to change the `root` user password.

> [!WARNING]
> I don't recommend you to allow to log in as the root user. It's a security vulnerability. You should be able to get root with your user account but not to log in directly into it. I won't describe the way to achieve this in this tutorial but you should inform yourself about that.

You can also create your user:

- `useradd -G wheel -s /bin/zsh -m -U [USERNAME]` **[USERNAME]** being what you want.

Now, let's install the microcode updater:

- `pacman -S intel-ucode` if you have an Intel CPU ;
- `pacman -S amd-ucode` if you have an AMD CPU.

### Bootloader 

There are two major choices for the bootloader:

- GRUB2 ;
- rEFInd.

I personnaly prefer Refind as it's easier to configure and customize.

To install it: 

- `pacman -S refind`.
- Then `refind-install`.

Now, to be able to boot into your system, you have to do this:

- `echo "\"Arch Linux\" \"root=UUID=$(blkid -o value -s UUID /dev/sdX) > /boot/refind\_linux.conf\"` where *sdX* is your root partition.
- `echo resume=UUID=$(blkid -o value -s UUID /dev/sdX) >> /boot/refind\_linux.conf` where *sdX* is your swap partition (not necessary if you don't have a swap partition).


