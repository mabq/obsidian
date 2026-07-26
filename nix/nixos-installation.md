# NixOS installation

### No disko

[[filesystems#^d53876|Filesystem UUIDs]] change every time you install (or re-install) NixOS, regardless if you use the graphical installer or not. Additionally, you cannot guarantee the same partition layout or filesystem types will be used when doing a full re-install.

In consequence, you need to:

1. Do a basic NixOS install — this will create the user account and home dir:
	- [Graphical Installation](https://nixos.org/manual/nixos/stable/#sec-installation-graphical) — good enough when no special partitioning is required.
	- [[#^70a55c|Manual Installation]] — when a special partition layout is required. Edit `configuration.nix` so that you can connect to the internet and the user account is created.
2. Reboot and re-login.
3. Run `nix shell "nixpkgs#git" "nixpkgs#gh" "nixpkgs#neovim" "nixpkgs#yazi"`.
4. Login to GitHub using `gh`.
	- Do I really need to login to clone a public repo?
5. Clone the repository to `~/.local/share/nixos-config`.
6.  the generated `hardware-configuration.nix` file into the repository's `hwconfig` directory.
	- Change its name to something that lets you identify the machine it belongs to.
	- One machine may have many hardware configuration files — one as the result of a manual installation, another one used for automated installations with `nixos-anywhere`, and so on.
	- Must be separated from the profile, otherwise you wouldn't be able to use the same profile on different machines.
7. Edit the flake, so that the desired configuration points to the right hardware configuration file.
8. Run `git add .` to add the new hardware configuration file to git.
9. Rebuild.

---
Basic installation
- NixOS is installed.
- The user account and directory (`/home/<USER>`) exist.
- Can connect to the internet
---

### Manual Installation

^70a55c

Download the [NixOS Minimal ISO image](https://nixos.org/download/#nixos-iso) and create a bootable USB drive following the instructions in [Booting from a USB flash drive](https://nixos.org/manual/nixos/stable/index.html#sec-booting-from-usb) section of the NixOS manual. Boot the machine from this USB drive.

> [!info]
> For more information see [Manual Installation](https://nixos.org/manual/nixos/stable/#sec-installation-manual) (NixOS Manual).

```sh
# Login as root
sudo -i

# Connect to a wireless network (or just plug the ethernet cable)
nmtui
# Verify the connection
ping 8.8.8.8

# Optionally, to continue via ssh (`ssh root@<IP>`)
# Change root's password
passwd
# Annotate the IP address
ip a

# Identify the target disk
lsblk -f

# ⚠️ From here on, replace `/dev/sdX` with the target disk device node. Be careful!

# Wipe all previous filesystems
wipefs -a /dev/sdX[0-9]* # careful!
# Wipe previous partition table
wipefs -a /dev/sdX # careful!
```

Partitioning, formatting and mounting (choose one of the options):

- BIOS (ext4 encrypted):

```sh
# Create GPT partition table
parted -s /dev/sdX mklabel gpt # careful!
# Partition 1
# BIOS boot partition (1MiB size, flagged for bios_grub)
parted -s /dev/sdX mkpart primary 1MiB 2MiB
parted -s /dev/sdX set 1 bios_grub on
# Partition 2
# Boot partition (1GiB for kernels/initrd)
parted -s /dev/sdX mkpart primary ext4 2MiB 1026MiB
# Partition 3
# Root partition (Rest of disk)
parted -s /dev/sdX mkpart primary ext4 1026MiB 100%
# Verify partition layout
parted /dev/sdX print

# Encrypt root partition
cryptsetup luksFormat /dev/sdX3
# Open encrypted partition (Enter decryption password when prompted)
cryptsetup open /dev/sdX3 cryptroot

# Create filesystems
# (BIOS partition does not requiere a filesystem)
mkfs.ext4 -L boot /dev/sdX2
mkfs.ext4 -L root /dev/mapper/cryptroot

# Mount filesystem
mount /dev/disk/by-label/root /mnt
mkdir -p /mnt/boot
mount /dev/disk/by-label/boot /mnt/boot

```

- For UEFI

```sh
lsblk -f        # Identify the target disk

parted /dev/sdX -- mklabel gpt
parted -s /dev/sdX mkpart primary 1MiB 2MiB      # BIOS partition
parted -s /dev/sdX set 1 bios_grub on
parted -s /dev/sdX mkpart primary 2MiB 514MiB    # UEFI partitition
parted -s /dev/sdX set 2 esp on
parted -s /dev/sdX mkpart primary 514MiB 100%    # root partition

# Encryption
cryptsetup luksFormat /dev/sdX3
cryptsetup open /dev/sdX3 cryptroot    # Enter decryption password when prompted

# Create filesystems
mkfs.vfat -F32 -n boot /dev/sdX2            # UEFI requires FAT32 (label "boot")
mkfs.ext4 -L nixos /dev/mapper/cryptroot    # ext4 for root (label "nixos")
                                            # BIOS requires no filesystem

# Mount filesystems
mount /dev/disk/by-label/nixos /mnt                     # Mount root
mkdir -p /mnt/boot                                      # Mount boot
mount -o umask=077 /dev/disk/by-label/boot /mnt/boot
```

Generate and edit configuration files:

```sh
# Generate config files
nixos-generate-config --root /mnt

# Edit configuration.nix
nano /mnt/etc/nixos/configuration.nix
# - BIOS? Uncomment the line `boot.loader.grub.device`
# - Uncomment the user block
#   - Change the username
#   - Add the `networkmanager` group
# - Enable ssh
# - Disable firewall
```

Install NixOS:

```sh
# Install NixOS (enter password for root when prompted)
nixos-install

# Set the user password
nixos-enter --root /mnt -c 'passwd <USER>'

# Reboot
reboot
```

Clone flake repo, add hardware configuration and rebuild:

```sh
# Install these tools in a temp shell
nix --extra-experimental-features "nix-command flakes" shell "nixpkgs#git" "nixpkgs#neovim" "nixpkgs#yazi"

# Clone the flake repository
mkdir -p ~/.local/share
cd ~/.local/share
git clone https://github.com/mabq/nixos-config.git
cd nixos-config

# Optionally, checkout the desired branch
git chechout <BRANCH>

# Move the generated hardware configuration file to the repository
sudo mv /etc/nixos/hardware-configuration.nix ~/.local/share/nixos-config/machines/hardware-configuration/<MACHINE-REF>-<YYYYMMDD>.nix

# Add new files to git
git add .

# Make sure the options passed to the nixos-configuration in the flake file
# are valid, then execute.
**sudo** nixos-rebuild --verbose switch --flake .#<CONFIG>

# Authenticate to GitHub
# ✋ Requieres another device logged into GitHub
gh auth login

# Push changes
git commit -m "hardware config <SOME REFERENCE>"
git push
```







All you need is network manager to connect to internet and the user account.