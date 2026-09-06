# Sops

[SOPS](https://github.com/getsops/sops) (Secrets Operations) is an editor of encrypted files that supports YAML, JSON, ENV, INI and BINARY formats and encrypts with AWS KMS, GCP KMS, Azure Key Vault, HuaweiCloud KMS, [[age]], and PGP.

> [!info]
> SOPS [recommends using age](https://getsops.io/docs/usage/identities/age/) over PGP. [[age#^09f97e|here]].

Encrypted files can be securely stored in public repositories.


When you execute `sops <file>`, it:

  - Reads `.sops.yaml` to map a public key to the file.
  - Verifies the matching private key exists in `~/.config/sops/age/keys.txt`.
  - Decrypts the file using the private key.
  - Creates a temporary file with the decrypted content and automatically opens it (`$EDITOR`).
  - Automatically re-encrypts the file on save, using the public key.

> [!info]
> `sops` expects to find the keys in `~/.config/sops/age/keys.txt`


### sops-nix

[sops-nix](https://github.com/mic92/sops-nix) provides a way to integrate sops with Nix/NixOS — during activation time `sops-nix` will:

  - Read your NixOS options (e.g., `sops.defaultSopsFile` and `sops.age.keyFile`).
  - Open the encrypted file (e.g., `secrets/<USER>.yaml`).
  - Extract the public key IDs directly from the file's internal metadata, ignoring `.sops.yaml` completely. 
  - Uses your host's local private key to perform the decryption and places the secret into `/run/secrets/<secret-attribute>`.  


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
5. 