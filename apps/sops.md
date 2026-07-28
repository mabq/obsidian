# Sops

[SOPS](https://github.com/getsops/sops) (Secrets Operations) is an editor that lets you edit encrypted files (YAML, JSON, ENV, INI, or binary) as if they were normal files — avoiding the need to manually decrypt → edit → re-encrypt secret files.

Encrypted files can be securely stored in public repositories.

When you execute `sops <file>`, it:

  - Maps the file to a public key based on the rules described in the `.sops.yaml` file.
  - Verifies the public key has a matching private key (`~/.config/sops/age/keys.txt`).
  - Decrypts the file temporarily using the private key.
  - Opens the cleartext version in your normal text editor (`$EDITOR`).
  - Re-encrypts the file automatically when saving, using the public key.


### Public/private key pair

SOPS [recommends](https://getsops.io/docs/usage/identities/age/) using [[age]] over PGP to encrypt files.

Asymetric encryption requieres a public/private key pair — you can easily [[age#^09f97e|create one]] if you don't have one already.

> [!info]
> The security model of managing secrets in a public repository relies entirely on the fact that an attacker cannot do anything with the encrypted blobs because they lack the private key — to facilitate things I included all private keys (simetrically encrypted) in the public repository.
>
> After cloning the repo you need to manually execute:
> ```sh
> cd ~/.config/sops/age
> age --decrypt -o keys.txt keys.txt.age
> ```
> ⚠️ MAKE SURE THE DECRYPTED FILE NEVER LEAKS THE REPOSITORY!

TODO: This won't be possible, first clone the repo, then 

### sops-nix

[sops-nix](https://github.com/mic92/sops-nix) decrypts secrets from soap files during activation time — you can use the decrypted content (stored securely in memory) on .



