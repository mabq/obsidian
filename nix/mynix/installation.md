# Installation


## With `nixos-anywhere`

The host must be reachable over a wired network. Review [requirements](https://nix-community.github.io/nixos-anywhere/#requirements).

On the target host:

```sh
# Boot from the ISO

# Connect the ethernet cable and check the connection (wireless not supported)
ping 8.8.8.8

# Change root's password
sudo -i # login as root (no password)
passwd # enter new password when prompted

# Get host's details
ip a # ip address
lsblk -o NAME,ID-LINK # wwn id of target disk
nixos-version # installer version
```

On the workstation:

```sh
# I assume the workstation: 1) is logged into GitHub, 2) has cloned the
# repository, 3) has the secret's private key in place.

# Cd into the flake directory
cd ~/.local/share/mynix
git checkout <branch>

# Open repo and review configs.
vim .
# - In the host file, pay special attention to `disko.devices.disk.main.device`
#   and `system.stateVersion`.
# - If secrets are required, create/edit the secrets file with `sops`.

# Close vim and update the flake
nix flake update

# Prepare "extra-files" directory (omit if secrets are not required)
temp=$(mktemp -d)
pathToKey="$temp/var/lib/sops-nix"
install -d -m755 "$pathToKey"
sudo cp /var/lib/sops-nix/key.txt "$pathToKey/key.txt"
sudo chmod 600 "$pathToKey/key.txt"

# Install NixOS on the remote host with `nixos-anywhere`
#  (`sudo` is required to read the private key)
#  (omit `--extra-files` line if you dont pass secrets)
#  (omit `--generate-hardware-config` if `facter.json` already exist)
sudo nix run github:nix-community/nixos-anywhere -- \
  --extra-files "$temp" \
  --generate-hardware-config nixos-facter hosts/facter/<HOST>.json \
  --flake .#<NIXOS-CONFIGURATION> \
  --target-host root@<IP>
```
  

## With `disko`

### With your workstation

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

```sh
# I assume the workstation: 1) is logged into GitHub, 2) has cloned the
# repository, 3) has the secret's private key in place.

# Open two terminals. One will be used exclusively to execute commands on the
# host via ssh, the other one to execute commands on the local workstation.


# (SSH TERMINAL)

# ssh into the target host
ssh root@<HOST-IP> # enter password when prompted

# Get host's details
lsblk -o NAME,ID-LINK # wwn id of target disk
nixos-version # installer version

# Generate the facter report
nix --experimental-features "nix-command flakes" run "nixpkgs#nixos-facter" -- -o /tmp/<HOST>.json


# (WORKSTATION TERMINAL)

# Cd into the flake directory
cd ~/.local/share/mynix
git checkout <branch>

# Copy the facter report from the host to the workstation
scp root@<HOST-IP>:/tmp/<HOST>.json hosts/facter/<HOST>.json

# Open repo and review configs.
vim .
# - In the host file, pay special attention to `disko.devices.disk.main.device`
#   and `system.stateVersion`.
# - If secrets are required, create/edit the secrets file with `sops`.

# Close vim and update the flake
nix flake update

# Once all configurations are ready, commit and push (use lazygit or):
git status
git add .
git commit -m "<MESSAGE>"
git push


# (SSH TERMINAL)

# Clone the repository (public)
git clone https://github.com/mabq/mynix
cd mynix
git checkout <branch> # (optionally)

# Run disko (destroy, partition, format and mount)
# NOTE: If you get an error here mentioning lack of space; reboot the
# machine, clone the repo again an re-run.
nix --extra-experimental-features "nix-command flakes" run 'github:nix-community/disko/latest' -- --mode disko --flake .#<NIXOS-CONFIG>
  
# Review partitions are mounted (`/mnt` and `/mnt/boot`)
lsblk

# Create dir for private key
mkdir -p /mnt/var/lib/sops-nix

 
# (WORKSTATION TERMINAL)

# If secrets apply, copy the private key from the workstation to the host.
# The key is owned by root, so you need to use `sudo`.
sudo scp -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null /var/lib/sops-nix/key.txt root@<IP>:/mnt/var/lib/sops-nix/key.txt
 
 
# (SSH TERMINAL)

# Ensure private key is only readable by root
chmod 600 /mnt/var/lib/sops-nix/key.txt

# Install
nixos-install --flake .#<NIXOS-CONFIG>
```


### Without your workstation

This method requieres authenticating to GitHub (2FA device or recovery code).

```sh
# Boot from the ISO...

# Connect to internet (ethernet or `nmtui`)
ping 8.8.8.8 # test the connection

# Annotate the following host's details
ip a # ip address
lsblk -o NAME,ID-LINK # wwn id of the target disk
nixos-version # installer version

# Continue as root
sudo -i # login as root (no password)
passwd # change its password

# Continue via ssh from another machine (`ssh root@<IP>`)

# Clone the repository
git clone https://github.com/mabq/mynix
cd mynix
git checkout <branch>

# Configure git (required to add files to git, files not added to git are 
# invisible to nix).
git config --global user.email "<EMAIL>"
git config --global user.name "<NAME>"

# Authenticate with gh to be able to push changes
nix --experimental-features "nix-command flakes" shell nixpkgs#gh
gh auth login

# -----------------------------------------------------------------------------
# Generate a `facter.json` report
# -----------------------------------------------------------------------------

# If this is the first time configuring this host (or if you want to update
# its facter report) execute the following command.
#
# A facter.json report provides richer, more flexible hardware detection: it
# dumps a detailed machine-readable JSON report that NixOS modules interpret
# to auto-enable drivers, kernel modules, graphics, networking, firmware, etc.
# Instead of baking fixed choices into Nix code (`hardware-configuration.nix`).
#
# Disko takes care of all file system configurations.
nix --experimental-features "nix-command flakes" run "nixpkgs#nixos-facter" -- -o hosts/facter/<HOST>.json

# (Use as fallback)
# Just in case you prefer the old approach, this is the command to # execute
# to generate a hardware configuration file.
nixos-generate-config --no-filesystems --root /mnt
mv /mnt/etc/nixos/hardware-configuration.nix hosts/hardware-configuration/<HOST>.nix

# -----------------------------------------------------------------------------
# Disk setup with Disko
# -----------------------------------------------------------------------------

# Open the host file and make sure it imports the desired disko configuration.
# Update `disko.devices.disk.main.device` and `system.stateVersion` with the
# values obtained previously.
vim hosts/<HOST>.nix

# Add files to git to make them visible to nix
git status
git add .
git commit -m "Facter report for <HOST>"

# Run disko (destroy, partition, format and mount)
nix --extra-experimental-features "nix-command flakes" \
  run 'github:nix-community/disko/latest' -- \
  --mode disko \
  --flake .#<NIXOS-CONFIG>
  
# Review
lsblk # partitions should appear mounted in `/mnt` and `/mnt/boot`

# -----------------------------------------------------------------------------
# Secrets
# -----------------------------------------------------------------------------

# If your configuration included a secrets file, put the key in place so
# that secrets are decrypted during installation.

nix --experimental-features "nix-command flakes" shell nixpkgs#age
age -d -o /mnt/var/lib/sops-nix/key.txt /config/sops/<USER>/keys.txt.age
chmod 600 /mnt/var/lib/sops-nix/key.txt

# -----------------------------------------------------------------------------
# Push changes upstream
# -----------------------------------------------------------------------------
git status
git add .
git commit -m "<MESSAGE>"
git push

# Install
# -----------------------------------------------------------------------------
nixos-install --flake .#<NIXOS-CONFIG>
```
