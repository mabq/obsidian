# Sops

[SOPS](https://github.com/getsops/sops) (Secrets Operations) is an editor that lets you edit encrypted files (YAML, JSON, ENV, INI, or binary) as if they were normal files — avoiding the need to manually decrypt → edit → re-encrypt secret files.

Encrypted files can be securely stored in public repositories.

When you execute `sops <file>`, it:

  - Read `.sops.yaml` to map a public key to the file.
  - Verifies the matching private key exists in `~/.config/sops/age/keys.txt`.
  - Decrypts the file temporarily using the private key.
  - Opens the cleartext version in your normal text editor (`$EDITOR`).
  - Re-encrypts the file automatically when saving, using the public key.

SOPS [recommends](https://getsops.io/docs/usage/identities/age/) using [[age]] over PGP to encrypt files — if you don't have a public/private key pair yet, [[age#^09f97e|create one]].

> [!info]
> `sops` expects to find the private key in `~/.config/sops/age/keys.txt`


### sops-nix

[sops-nix](https://github.com/mic92/sops-nix) provides a way to integrate sops with Nix/NixOS — during activation time `sops-nix`:

  - Reads your NixOS options (e.g., `sops.defaultSopsFile` and `sops.age.keyFile`).
  - Opens the encrypted file (e.g., `secrets/<USER>.yaml`).
  - Extracts the public key IDs directly from the file's internal metadata, ignoring `.sops.yaml` completely. 
  - Uses your host's local private key to perform the decryption and places the secret into `/run/secrets/<secret-attribute>`.  

### nixos-anywhere

`nixos-anywhere` includes built-in options specifically designed to inject secrets (including your age private key) onto the target machine's disk after formatting (via disko), but before running `nixos-install`.

```sh
nixos-anywhere --extra-files /tmp/extra-files --flake .#myhost root@<TARGET_IP>
```

