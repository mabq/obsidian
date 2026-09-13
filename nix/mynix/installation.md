# NixOS Installation


## nixos-anywhere

**On the target machine:**

- Boot from ISO
- Check internet access (wireless not supported): `ping 8.8.8.8`
- Login as root: `sudo -i`
- Change password: `passwd`
- Annotate the ip address: `ip a`
- Annotate the wwn id of the target disk: `lsblk -o NAME,ID-LINK`
- Annotate the nixos version of the installer: `nixos-version`
- (continue from the source machine...)

**On the source machine:**

- Clone the repo containing the flake (public) and `cd` into it. 
- Checkout the desired branch (if not `main`): `git checkout <branch>`.
- Add / review the following files:
  - `flake.nix` - must point to the correct `host`, `user`, `profile` and `repoBranch` (if not `main`).
  - `/hosts/<host>.nix` -  make sure you replace the option values with the values you annotated previously from the target machine.
  - `/users/<user>.nix` - make sure your include an open ssh authorized key to avoid loosing access after installation.
  - `/profiles/<profile>.nix` - make sure the file exists and contains the configurations you need.
  - `/secrets/<user>-<host>-<profile>.json` - if you want to pass secrets
- Execute:
  ```sh
  # ---------------------------
  # If you want to pass secrets
  # ---------------------------
  # Create a temporary directory
  temp=$(mktemp -d)
  # Recreate the host path where sops-nix expects to find the key
  pathToKey="$temp/var/lib/sops-nix"
  install -d -m755 "$pathToKey"
  # Copy the key and make it readable only by its user
  age -d -o "$pathToKey/key.txt" ~/.local/share/mynix/config/sops/<USER>/keys.txt.age
  sudo chmod 600 "$pathToKey/key.txt"
  
  # --------------------------------
  # Install NixOS on the target host
  # --------------------------------
  # Omit `--extra-files` line if you dont use secrets.
  # Omit `--generate-hardware-config` if the `facter.json` report already exists.
  
  nix run github:nix-community/nixos-anywhere -- \
    --extra-files "$temp" \
    --generate-hardware-config nixos-facter hosts/facter/<host>.json \
    --flake .#<nixos-configuration> \
    --target-host root@<ip>
  ```
  

### From the device

> [!note]
> You need access to your GitHub account. If you don't have another device at hand, use the Graphical ISO to access your password manager via a browser (still need the 2FA device).

- Connect to internet. For wireless networks use `nmtui`.
- Login as root: `sudo -i`.
- Continue over ssh (optional). Run `passwd` to change root's password.
- Enable nix features: `export NIX_CONFIG="experimental-features = nix-command flakes"`.
- Install packages with: `nix shell nixpkgs#{gh,neovim,age,yazi}`.
- Authenticate to GitHub: `gh auth login`.
- Clone the repo: `gh repo clone mabq/mynix`.
- Change directory: `cd mynix`.
- Checkout the branch you want: `git checkout <branck>`
- Generate a facter report: `nix run nixpkgs#nixos-facter -- -o ./hosts/facter/<host>.json`
- Decrypt the private age key into a tmp file: `age -d -o /tmp/keys.txt secrets/<user>/keys.txt.age`. IMPORTANT! Be very careful not to put the decrypted key in a directory inside the cloned repository.
- Open the project `nvim .` and check that everything is in place - specially the options in the targeted host file.
- Verify changes with `git status`.
- Add, commit and push changes upstream: `git add .`, `git commit -m "..."` and `git push`.
- Run disko, using the configuration inside the flake: `sudo nix --experimental-features "nix-command flakes" run github:nix-community/disko#disko-install -- --flake .#<nixos-config>`.

```sh
sudo nix --extra-experimental-features "nix-command flakes" \
  run github:nix-community/disko/latest#disko-install -- \
  --flake ".#<your-config>" \
  --extra-files "/tmp/keys.txt" "/home/john/.config/sops/age/keys.txt" \
  --write-efi-boot-entries
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
