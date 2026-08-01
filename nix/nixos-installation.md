# NixOS Installation


### nixos-anywhere

Follow the instructions below — for more information see [Quickstart](https://nix-community.github.io/nixos-anywhere/quickstart.html).

> [!info]
> The target machine needs to be reachable via SSH as root or a user with password-less `sudo` — review the [requirements](https://nix-community.github.io/nixos-anywhere/requirements.html).

Before using `nixos-anywhere`, review:

1. Values of the desired nixos-configuration in `flake.nix`.
2. Values of the default module — use the profile file to override any.
3. The referenced host file exist and imports a valid [disko](https://github.com/nix-community/disko?tab=readme-ov-file#how-to-use-disko) configuration.
   Update the name of the referenced facter report to match the new report (created below) — e.g. `xps-20260729`.

Install NixOS remotely:

> [!info]
> nixos-anywhere doesn’t need to be installed, run it directly from the Github repository

```sh
nix run github:nix-community/nixos-anywhere -- \
  --flake <path/to/flake>#<nixos-configuration> \
  --generate-hardware-config nixos-generate-config ./hosts/hardware-configuration/<host>-<yyyymmdd>.nix
  --generate-hardware-config nixos-facter ./hosts/facter/<host>-<yyyymmdd>.json
  --target-host root@<ip address>
```

---

Then, on the **local machine**, prepare the extra files directory:

```sh
# nixos-anywhere will copy the contents of the `/tmp/extra-files` directory onto
# the target root filesystem (`/`) before system evaluation.
mkdir -p /tmp/extra-files/home/<USER>/.config/sops/age

# Store the user's decrypted private key file in the path where we need it on the
# remote machine — without the private key there sops-nix won't be able to decrypt
# secrets.
age --decrypt \
  -o /tmp/extra-files/home/<USER>/.config/sops/age/keys.txt \
  ~/.local/share/nixos-config/users/<USER>/keys.txt.age

# Ensure strict permissions
chmod 600 /tmp/extra-files/home/<USER>/.config/sops/age/keys.txt
chmod 700 /tmp/extra-files/home/<USER>/.config/sops/age
```

Run `nixos-anywhere` with `--extra-files`:

```sh
nixos-anywhere \
  --extra-files /tmp/extra-files \
  --flake .#<NIXOS-CONFIGURATION> \
  root@<TARGET_IP>
```


### Manual Installation

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
8. Decrypt user's age private key

	```sh
	# Make sure you target the "mounted" filesystem, not the live ISO.
	mkdir /mnt/home/<USER>/.config/sops/age
	# Careful about where you put the decrypted private key ⚠️
	age --decrypt -o /mnt/home/<USER>/.config/sops/age/keys.txt /mnt/home/<USER>/.local/share/nixos-config/users/<USER>/keys.txt.age
	# Should only be readable by the owner!
	chmod 600 /mnt/home/<USER>/.config/sops/age/keys.txt
	```

Publishing the encrypted key alongside the flake is no worse than publishing any other ciphertext. There's no rate-limiting on offline brute force since anyone can copy the file, so don't get lazy on entropy — treat it like a disk-encryption passphrase, not a login password.

---

[[filesystems#^d53876|Filesystem UUIDs]] change every time you install (or re-install) NixOS, regardless if you use the graphical installer or not. Additionally, you cannot guarantee the same partition layout or filesystem types will be used when doing a full re-install.

---

9. Run `git add .` to add the new hardware configuration file to git.
10. Rebuild.

---

Basic installation
- NixOS is installed.
- The user account and directory (`/home/<USER>`) exist.
- Can connect to the internet







All you need is network manager to connect to internet and the user account.