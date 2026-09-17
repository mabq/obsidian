# Installation

In the instructions below, the step that says "Open the repository and review the configurations" means reviewing the following files:

- `flake.nix`
  Reviewing the name of the nixos-configuration and the target host, user, profile, etc.
  <br>
- `hosts/<HOST>.nix`
  Reviewing that this file exists and ensure that 1) `disko.devices.disk.main.device` points to the correct target disk, 2) `system.stateVersion` matches the version of the installer and 3) that either `hardware.facter.reportPath` points to the facter report that we create in the steps below or that a `hardware-configuration.nix` file is imported instead.
<br>
- `secrets/<USER>-<HOST>-<PROFILE>.nix`
  Reviewing that this file exists (if secrets are required by the configuration). You will need to create/edit this file in advanced in a workstation where `sops` is configured.
<br>
- `users/<USER>.nix`
  Reviewing that this file exist. Normally this file is reused in all configs and does not need to be edited.
<br>
- `profiles/<PROFILE>.nix`
  Reviewing that this file exists and contains all the desired configurations.


## When the host is reachable over ethernet

When these [requirements](https://nix-community.github.io/nixos-anywhere/#requirements) are met, you can use `nixos-anywhere` to automate the installation.

On the target host:

```sh
# Boot from the ISO...

# Test ethernet connection
ping 8.8.8.8

# Change root's password on the live-environment
sudo -i # login as root (no password)
passwd # enter new password when prompted

# Annotate the following details
ip a # ip address
lsblk -o NAME,ID-LINK # wwn id of target disk
nixos-version # installer version
```

On the workstation:

```sh
# Clone the repo (if not already there already)
cd ~/.local/share/mynix
git clone https://github.com/mabq/mynix
cd mynix
git checkout <branch>

# Open the repository and review the configurations
vim .

# Once everything is set, update the lock file
nix flake update

# Commit and push (I assume the workstation is already authenticated to GitHub)
git status
git add .
git commit -m "<MESSAGE>"
git pushGitHub):

# Only if secrets are required by the targeted configuration, prepare
# the "extra-files" directory. I assume the private key is already in
# `/var/lib/sops-nix/key.txt`.
temp=$(mktemp -d)
pathToKey="$temp/var/lib/sops-nix"
install -d -m755 "$pathToKey"
sudo cp /var/lib/sops-nix/key.txt "$pathToKey/key.txt"
sudo chmod 600 "$pathToKey/key.txt"

# Install NixOS on the remote host
# `sudo` is required to read the private key.
# Omit the `--extra-files` line if you don't pass secrets.
# Omit the `--generate-hardware-config` line if the `facter.json` report already exist.
sudo nix run github:nix-community/nixos-anywhere -- \
  --extra-files "$temp" \
  --generate-hardware-config nixos-facter hosts/facter/<HOST>.json \
  --flake .#<NIXOS-CONFIGURATION> \
  --target-host root@<IP>
```
  

## When the host is not reachable over ethernet but you have a workstation with you

On the target host:

```sh
# Boot from the ISO...

# Change root's password
sudo -i # login as root (no password)
passwd # enter new password when prompted

# Connect to internet (ethernet or `nmtui`)
ip a # get the ip address
```

On the workstation:

> If the `facter.json` report already exist in the repo, you can skip most of the steps.

```sh
# ssh into the target host
ssh root@<HOST-IP> # enter password when prompted

# (SSH TERMINAL)

# Annotate the following details
lsblk -o NAME,ID-LINK # wwn id of target disk
nixos-version # installer version

# Generate the facter report
nix --experimental-features "nix-command flakes" \
  run "nixpkgs#nixos-facter" -- \
  -o /tmp/<HOST>.json

# (WORKSTATION TERMINAL)

# Clone the repo (if not already there already)
cd ~/.local/share/mynix
git clone https://github.com/mabq/mynix
cd mynix
git checkout <branch>

# Copy the facter report from the host to the workstation
scp root@<HOST-IP>:/tmp/<HOST>.json hosts/facter/<HOST>.json

# Open and review the configurations (see notes at the top of this file)
vim .

# Once everything is set, update the lock file
nix flake update

# Commit and push (I assume the workstation is already authenticated to GitHub)
git status
git add .
git commit -m "<MESSAGE>"
git push

# (SSH TERMINAL)

# Clone the repository (public, no need to push changes)
git clone https://github.com/mabq/mynix
cd mynix
git checkout <branch>

# Run disko (destroy, partition, format and mount)
# NOTE: If you get an error here mentioning lack of space; reboot the
# machine, clone the repo again an re-execute this command (everything
# on the live-environment is stored in memory, so by restarting the
# machine we free memory used in previous steps, the important thing is
# that the facter report is already in the repo).
nix --extra-experimental-features "nix-command flakes" \
  run 'github:nix-community/disko/latest' -- \
  --mode disko \
  --flake .#<NIXOS-CONFIG>
  
# Check if the new partitions are mounted (`/mnt` and `/mnt/boot`)
lsblk

# Create the directory for private key (if secrets are required)
mkdir -p /mnt/var/lib/sops-nix

# (WORKSTATION TERMINAL)

# If secrets are required, copy the private key from the workstation
# to the host. I assume the private key is already in place. Since the
# key is owned by root, you need to use `sudo`.
sudo scp \
  -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null \
  /var/lib/sops-nix/key.txt root@<IP>:/mnt/var/lib/sops-nix/key.txt
 
# (SSH TERMINAL)

# Ensure private key is only readable by root
chmod 600 /mnt/var/lib/sops-nix/key.txt

# Install
nixos-install --flake .#<NIXOS-CONFIG>

# If you get an error, reboot the machine and execute
cryptsetup open /dev/sdXN crypted # to open the encrypted partition
mount /dev/mapper/crypted /mnt
mkdir -p /mnt/boot
mount /dev/disk/by-partlabel/disk-main-ESP /mnt/boot
# Download the flake repo again
git clone https://github.com/mabq/mynix
cd mynix
git checkout <branch>
# Try to install, again
nixos-install --flake .#<NIXOS-CONFIG>
```


## When you don't have a workstation

You will still need access to GitHub's 2FA device (if you don't, use the graphical installer and the recovery codes).

```sh
# Boot from the ISO...

# Connect to internet (ethernet or `nmtui`) and test the connection
ping 8.8.8.8

# Annotate the following details
ip a # ip address
lsblk -o NAME,ID-LINK # wwn id of the target disk
nixos-version # installer version

# Continue as root
sudo -i # login as root (no password)
passwd # change its password

# Install required tools in the live-environment
nix --experimental-features "nix-command flakes" shell nixpkgs#{gh,age}

# If this is a new host, authenticate to GitHub (we need to push changes)
gh auth login
git config --global user.email "<EMAIL>" # required to push changes
git config --global user.name "<NAME>" # required to push changes

# Clone the flake repo (public)
git clone https://github.com/mabq/mynix
cd mynix
git checkout <branch>

# Generate the `facter.json` report (if it does not exist yet)
nix --experimental-features "nix-command flakes" run "nixpkgs#nixos-facter" -- -o hosts/facter/<HOST>.json

# Open and review the configurations (see above)
vim .

# Once everything is set, commit and push
git status
git add .
git commit -m "Facter report for <HOST>"
git push

# Run disko (destroy, partition, format and mount)
# NOTE: If you get an error here mentioning lack of space; reboot the
# machine, clone the repo again an re-execute this command.
nix --extra-experimental-features "nix-command flakes" \
  run 'github:nix-community/disko/latest' -- \
  --mode disko \
  --flake .#<NIXOS-CONFIG>
  
# Check if the new partitions are mounted (`/mnt` and `/mnt/boot`)
lsblk

# If secrets are required, decrypt the private key and make sure it
# is only readable by root.
age -d -o /mnt/var/lib/sops-nix/key.txt config/sops/<USER>/keys.txt.age
chmod 600 /mnt/var/lib/sops-nix/key.txt

# Install nixos
nixos-install --flake .#<NIXOS-CONFIG>
```
