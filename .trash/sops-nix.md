# sops-nix

[sops-nix](https://github.com/mic92/sops-nix) provides a way to integrate sops with Nix/NixOS — during activation time `sops-nix` will:

  - Read your NixOS options (e.g., `sops.defaultSopsFile` and `sops.age.keyFile`).
  - Open the encrypted file (e.g., `secrets/<USER>.yaml`).
  - Extract the public key IDs directly from the file's internal metadata, ignoring `.sops.yaml` completely. 
  - Uses your host's local private key to perform the decryption and places the secret into `/run/secrets/<secret-attribute>`.  


### Structure

My nixos configurations expect the following arguments:

- host
  
- user
- profile


### disko-install

1. The age key must be copied to `/etc/mynix/sops/age/keys.txt`. The owner of the file will be root, so you will need to uso `sops` with `sudo` anytime you want to edit a secrets file.

Check if disko-install can generate a facter report, if so you should clone the repo.

```sh
sudo nix run 'github:nix-community/disko#disko-install' -- \
  --flake '<url>#<config>' \
  # facter report?
  #--disk main /dev/sda \
  --extra-files /tmp/keys.txt /etc/mynix/sops/age/keys.txt
```

What permissions are set on the age key file?

The `--disk` flag is designed to override the device value in your flake, not to require it.



### nixos-anywhere

`nixos-anywhere` includes built-in options specifically designed to inject secrets (including your age private key) onto the target machine's disk after formatting (via disko), but before running `nixos-install`.

```sh
nixos-anywhere --extra-files /tmp/extra-files --flake .#myhost root@<TARGET_IP>
```



---

On the source machine:

1. Generate an ssh key pair
2. Transform the ssh public key into an age key with `ssh-to-age`.
3. Add the new generated public age key to `.sops.yaml`
4. Re-encrypt files with `sops updatekeys <path/to/secrets/file>.yaml` — I guess sops needs to decrypt the file first to encrypting it again, which private key does it use for that matter.
5

