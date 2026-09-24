# Installation

This document shows how to install NixOS with "mynix" flake and secrets — for complementary information review the following links:

- [Create a bootable USB drive](https://nixos.org/manual/nixos/stable/index.html#sec-booting-from-usb)
- [Graphical Installation Guide](https://nixos.org/manual/nixos/stable/#sec-installation-graphical)
- [Manual Installation Guide](https://nixos.org/manual/nixos/stable/#sec-installation-manual)

> [!note]
> Wherever you see "REVIEW FLAKE FILES" in the instruction below, it means you should review the following files:
>
> - `**flake.nix**`
>
>	Review the targeted `nixosConfiguration` and that all the files it points to actually exists.
>	<br>
>
> - **`hosts/<HOST>.nix`**
>
>	If the host uses disko + facter, make sure:
>	- The `imports` section imports disko modules and desired disk configuration.
>	- The option `disko.devices.disk.main.device` points to the correct target disk — use `lsblk -o NAME,ID-LINK` to check the target disk wwn id.
>	- The option `hardware.facter.reportPath` is enabled and points to the host facter report.
>	- The option `system.stateVersion` matches the version of the installer — use `nixos-version` to check the version of the installer.
>	- The options for GRUB/systemd-boot are enabled.
>
>	If the host uses a hardware configuration file, make sure:
>	- The `imports` section imports the hardware configuration file.
>	- No disko or facter options are enabled.
>	- The option `system.stateVersion` matches the version of the installer — use `nixos-version` to check the version of the installer.
>	- The options for GRUB/systemd-boot are enabled.
>	<br>
>	
> - `users/<USER>.nix`
>
>	Make sure `openssh.authorizedKeys.keys` includes your public ssh key to avoid loosing access after the build.
>	<br>
>	
> - `profiles/<PROFILE>.nix`
>
>	Ensure this file exists and imports/sets all the desired configurations.
>	<br>
>	
> - `secrets/<USER>-<HOST>-<PROFILE>.nix`
>
>	If you want to use secrets, ensure this file exists and contains the secrets you want. Note that this file must be created/edited with `sops` in a machine where the private key is in place.


## nixos-anywhere (disko) + facter.json

> [!info]
> This is the preferred option. The target host must be reachable over the public internet or local network — nixos-anywhere does not support wifi networks, review [requirements](https://nix-community.github.io/nixos-anywhere/#requirements).
>
> Requires access to GitHub's 2FA device (or that the workstation is already authenticated).

On the target host:

```sh
# Boot from the ISO...

# Test connection (must be ethernet)
ping 8.8.8.8

# Login as root and change its password
sudo -i
passwd

# Annotate host details (required later)
ip a # ip address
lsblk -o NAME,ID-LINK # wwn id of target disk
nixos-version # installer version
```

On the workstation:

```sh
# Clone the flake repo (if not already there)
git clone https://github.com/mabq/mynix.git
cd mynix
git checkout <branch>

# REVIEW FLAKE FILES (see notes on top)
vim .

# Update `flake.lock` (flake inputs)
nix flake update

# Push new files / changes
# (if not already authenticated, use `gh auth login` and the 2FA)
git status
git add .
git commit -m "<MESSAGE>"
git push

# If secrets are required, prepare the "extra-files" directory.
# I assume the private key is already in `/var/lib/sops-nix/key.txt`.
temp=$(mktemp -d)
pathToKey="$temp/var/lib/sops-nix"
install -d -m755 "$pathToKey"
sudo cp /var/lib/sops-nix/key.txt "$pathToKey/key.txt"
sudo chmod 600 "$pathToKey/key.txt"

# Build locally, install remotely
# `sudo` is required to read the private key (owned by root).
# Omit the `--extra-files` line if you don't pass secrets.
# Omit the `--generate-hardware-config` line if the `facter.json` report already exist.
sudo nix run github:nix-community/nixos-anywhere -- \
  --extra-files "$temp" \
  --generate-hardware-config nixos-facter hosts/facter/<HOST>.json \
  --flake .#<NIXOS-CONFIGURATION> \
  --target-host root@<IP>
# Enter root's password and LUKS passphrase when prompted

# If a facter report was generated, don't forget to push it upstream
git status
git add .
git commit -m "<HOST> facter report"
git push

# If everything goes well the host will reboot into the new system.
```
 

## Manual disk setup + hardware-configuration.nix

> [!info]
> This option does not require a workstation, but if you have one you can ssh into the target host to avoid manually typing all commands.
> 
> Requires access to the 2FA device (or a workstation that is already authenticated to GitHub).

Boot from the ISO...

```sh
# Check internet access (use `nmtui` for wireless)
ping 8.8.8.8

# Login as root
sudo -i

# Optionally, continue via `ssh root@<IP>`
passwd # change root's password
ip a
```

> [!note]
> If you are using ssh this would be a good moment to review flake files (see notes on top).

Set varibles (helps a lot when using ssh to avoid replacing values):

```sh
# Get host info
lsblk # target disk

# Set variables
DISK="/dev/<DISK>"
HOST="<HOST>"
USER="<USER>"
EMAIL="<EMAIL>"
```

Manual disk setup — compatible with BIOS (GRUB) and UEFI (systemd-boot):

```sh
# Optionally, wipe...
wipefs -a "$DISK"[0-9]*  # all previous filesystems
wipefs -a "$DISK"  # previous partition table

# --- Create disk partitions ---

# Create a new GPT table
parted -s "$DISK" -- mklabel gpt

# Partition names appear in `/dev/disk/by-partlabel/` and are used to make
# the hardware-configuration file reusable (when using this same layout).

# Partition 1: BIOS boot partition (MBR/GRUB), 1M (must be first)
parted -s "$DISK" -- mkpart disk-main-MBR 1MiB 2MiB
parted -s "$DISK" -- set 1 bios_grub on

# Partition 2: ESP, 500M (Extensible Firmware Interface System Partition / UEFI)
parted -s "$DISK" -- mkpart disk-main-ESP fat32 2MiB 502MiB
parted -s "$DISK" -- set 2 ESP on

# Partition 3: LUKS container, rest of disk
parted -s "$DISK" -- mkpart disk-main-luks 502MiB 100%

# --- Create file systems ---

# Partition 1:
#  No filesystem required. The BIOS boot partition isn't meant to hold files at
#  all — GRUB writes its `core.img` (the second-stage bootloader) directly to the
#  raw blocks of that partition, with no filesystem layer in between.

# Partition 2: Must be fat-32
mkfs.vfat -F32 -n ESP "${DISK}2"

# Partition 3: Create LUKS container (enter passphrase when prompted)
cryptsetup luksFormat "${DISK}3"
# Open encrypted container (enter passphrase when prompted)
# Note: `--allow-discards` exposes a very small security risk in exchange of
# extended life time and performance. Only applies to SSDs.
cryptsetup open --allow-discards "${DISK}3" crypted
# Format the opened luks container
mkfs.ext4 -L crypted /dev/mapper/crypted

# --- Mount file systems ---

# Mount root filesystem
mount /dev/mapper/crypted /mnt

# Mount boot filesystem (only readable by root)
mkdir -p /mnt/boot
mount -o umask=0077 "${DISK}2" /mnt/boot
```

Generate hardware configuration:

```sh
# Clone repo to put the hardware configuration inside it
git clone https://github.com/mabq/mynix.git
cd mynix
git checkout <branch>

# Create hardware config
nixos-generate-config --show-hardware-config --root /mnt > "hosts/hardware-configuration/${HOST}.nix"

# Optionally, replace disks UUIDs with part labels
#  crypted.device -> "/dev/disk/by-partlabel/disk-main-luks"
#  fileSystems."/boot" -> "/dev/disk/by-partlabel/disk-main-ESP"
vim "hosts/hardware-configuration/${HOST}.nix"

# Review the rest of flake config files (see notes on top)
vim .
```

Only if the targeted configuration requires secrets, put the private key in place:

```sh
# Get the age package
nix --experimental-features "nix-command flakes" shell nixpkgs#age

# Decrypt private key into place
mkdir -p /mnt/var/lib/sops-nix
age -d -o /mnt/var/lib/sops-nix/key.txt "config/sops/${USER}/keys.txt.age"
chmod 600 /mnt/var/lib/sops-nix/key.txt

# Check
ls -al /mnt/var/lib/sops-nix
cat /mnt/var/lib/sops-nix/key.txt

# Exit temporary shell
exit
```

Once everything is ready, save and push files upstream:

```sh
# Authenticate the GitHub
nix --experimental-features "nix-command flakes" shell nixpkgs#gh
gh auth login # (requires 2FA device)

git config --global user.email "${EMAIL}"
git config --global user.name "${USER}"

# Update `flake.lock` (flake inputs)
nix --experimental-features "nix-command flakes" flake update

# Add new files to git (files not added to git are invisible to flakes)
git status
git add .
git commit -m "Host ${HOST}"
git push

# Exit temporary shell
exit
```

Install:

```sh
nixos-install --flake ".#${HOST}"

# If everything went well
reboot

# You should now be able to boot into the installed NixOS
```


## disko + facter.json

> [!tip]
> This option actually takes longer than the manual disk setup above when using ssh.

> [!info]
> Use this option whenever you have your workstation but the target host is only reachable via Wi-Fi connection.
>
> Requires access to GitHub's 2FA device (or that the workstation is already authenticated).

On the target host:

```sh
# Boot from the ISO...

# Connect to internet and test
nmtui # connect to a wireless network
ping 8.8.8.8

# Login as root and change its password
sudo -i
passwd

# Get host ip address
ip a
```

On the workstation (ssh terminal):

```sh
# SSH into the target host
ssh root@<IP>

# Generate the facter report (disk configuration is managed by disko)
nix --experimental-features "nix-command flakes" \
  run "nixpkgs#nixos-facter" -- \
  -o facter.json 

# Get host info
lsblk -o NAME,ID-LINK # wwn id of target disk
nixos-version # installer version
```

On the workstation (non-ssh terminal)

```sh
# Clone the flake repo (if not already cloned)
git clone https://github.com/mabq/mynix.git
cd mynix
git checkout <branch>

# Copy facter report from target host
scp root@<IP>:facter.json hosts/facter/<HOST>.json

# REVIEW FLAKE FILES (see notes on top)
vim .

# Update `flake.lock` (flake inputs)
nix flake update

# Push all changes upstream
# (if not already authenticated, use `gh auth login` and the 2FA)
git status
git add .
git commit -m "<MESSAGE>"
git push
```

On the workstation (ssh terminal)

```sh
# Clone the flake repo
git clone https://github.com/mabq/mynix.git
cd mynix
git checkout <branch> 

# Setup disk with disko
nix --extra-experimental-features "nix-command flakes" \
  run github:nix-community/disko -- \
  --mode disko \
  --flake .#<NIXOS-CONFIG>
```

Only if the targeted configuration requires secrets, put the private key in place (ssh terminal):

```sh
# Get the age package
nix --experimental-features "nix-command flakes" shell nixpkgs#age

# Decrypt private key into place
mkdir -p /mnt/var/lib/sops-nix
age -d -o /mnt/var/lib/sops-nix/key.txt config/sops/<USER>/keys.txt.age
chmod 600 /mnt/var/lib/sops-nix/key.txt

# Check
ls -al /mnt/var/lib/sops-nix/key.txt

# Exit temporary shell
exit
```

Install:

```sh
nixos-install --flake .#<HOST>

# If everything went well
reboot

# You should now be able to boot into the installed NixOS
```
