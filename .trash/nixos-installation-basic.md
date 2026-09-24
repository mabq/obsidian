# NixOS Basic Installation

Create a basic NixOS installation by following the instructions below — for more information see notes about [[firmware]], [[partitions]], [[luks]] and [[filesystems]].

> [!info]
> Do a Manual Installation if you require some special disk partitioning layout, otherwise use the Graphical Installer.

### Graphical Installation

^21fac6

Download the [NixOS Graphical ISO image](https://nixos.org/download/#nixos-iso) and create a [bootable USB drive](https://nixos.org/manual/nixos/stable/index.html#sec-booting-from-usb).

Follow the guided Calamares install and reboot — for more information see [Graphical Installation](https://nixos.org/manual/nixos/stable/#sec-installation-graphical).


### Manual Installation

^70a55c

Download the [NixOS Minimal ISO image](https://nixos.org/download/#nixos-iso) and create a [bootable USB drive](https://nixos.org/manual/nixos/stable/index.html#sec-booting-from-usb).

Boot the machine from the USB drive and run the following commands — for more information see [Manual Installation](https://nixos.org/manual/nixos/stable/#sec-installation-manual):

```sh
# Login as root (no password)
sudo -i

# Connect to a wireless network (or plug the ethernet cable)
nmtui
ping 8.8.8.8  # Verify the connection

# Optionally, to continue via ssh (`ssh root@<IP>`)
passwd  # Change root's password temporarily
ip a  # Annotate the IP address
```

Disk setup:

> [!warning]
> Replace `/dev/sdX` with the device node for the target disk.
> Always double check device node before pressing Enter!

```sh
# Identify target disk device node
lsblk -d -o NAME,SIZE,MODEL

# Optionally, wipe previous data
wipefs -a /dev/sdX[0-9]*  # Wipe all previous filesystems
wipefs -a /dev/sdX  # Wipe previous partition table
```

- BIOS — ext4 encrypted:

```sh
# Create a new GPT partition table
parted -s /dev/sdX -- mklabel gpt
parted -s /dev/sdX -- mkpart primary 1mib 2mib  # Bios boot partition
parted -s /dev/sdX -- set 1 bios_grub on  # Flagged for bios_grub
parted -s /dev/sdX -- mkpart primary ext4 2mib 1026mib  # Boot partition (grub)
parted -s /dev/sdX -- mkpart primary ext4 1026mib 100%  # Root partition (rest of disk)
parted /dev/sdX -- print  # Verify partition layout

# Encrypt Root partition
cryptsetup luksformat /dev/sdX3  # Enter passphrase when prompted
cryptsetup open /dev/sdX3 cryptroot

# Create filesystems — except for the Bios partition
mkfs.ext4 -l boot /dev/sdX2
mkfs.ext4 -l cryptroot /dev/mapper/cryptroot

# Mount filesystem
mount /dev/disk/by-label/cryptroot /mnt
mkdir -p /mnt/boot
mount /dev/disk/by-label/boot /mnt/boot
```

- UEFI — ext4 encrypted:

```sh
# Create a new GPT partition table
parted -s /dev/sdX -- mklabel gpt
parted -s /dev/sdX -- mkpart ESP fat32 1MB 512MB  # Boot parition (Grub)
parted -s /dev/sdX -- set 1 esp on  # Flagged as EFI System Partition for UEFI
parted -s /dev/sdX -- mkpart primary ext4 512MB 100% # Root partition (rest of disk)
parted -s /dev/sdX -- print  # Verify partition layout

# Encrypt Root partition
cryptsetup luksformat /dev/sdX2  # Enter passphrase when prompted
cryptsetup open /dev/sdX2 cryptroot

# Create filesystems
mkfs.vfat -F 32 -n boot /dev/sdX1  # UEFI required FAT32
mkfs.ext4 -l cryptroot /dev/mapper/cryptroot

# Mount filesystem
mount /dev/disk/by-label/cryptroot /mnt
mkdir -p /mnt/boot
mount -o umask=077 /dev/disk/by-label/boot /mnt/boot  # Only readable by root
```

- UEFI — btrfs encrypted:

```sh
TODO...
```

Generate and edit NixOS configuration file:

```sh
# Generate config files
nixos-generate-config --root /mnt

# Basic configuration
nano /mnt/etc/nixos/configuration.nix
# - BIOS? Uncomment the line `boot.loader.grub.device`
# - Uncomment the user section and:
#   - Change the username
#   - Add the `networkmanager` group
# - Enable ssh
# - Disable firewall
```

Install NixOS:

```sh
# Install NixOS
nixos-install  # Enter password for root when prompted

# Set non-root password
nixos-enter --root /mnt -c 'passwd <USER>'  # Replace <USER>

# Reboot
reboot
```

Now, you should be able to login to your new NixOS system.