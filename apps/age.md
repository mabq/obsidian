# age

A simple, modern and secure encryption tool with small explicit keys, no config options, and UNIX-style composability.


### Symetric encrytion

^778d51

Age supports symetric encryption — use the following commands to encrypt/decrypt files using a passphrase:

> [!warning]
> Never type the passphrase inside an untrusted system — see  [[bitwarden#^990072|2FA]].

```sh
# Encrypt a file using ASCII-only "armored" encoding
# ⚠️ Use a long, random passphrase!
age --armor --passphrase -o <entrypted-file>.age <file-to-encrypt>

# Output decrypted content to file
# ⚠️ Make sure the cwd is not shared or tracked by git!
age --decrypt -o <output-file> <encrypted-file>.age

# Output decrypted content to stdout
age --decrypt <encrypted-file>.age
```


### Asymetric encryption

^09f97e

Age supports asymetric encryption — public key encrypts, private key decrypts.

Create a public/private key pair using the following command:

> [!warning]
> Loosing control of the private key means loosing everything!

```sh
# E.g. SOPS expects to find the `keys.txt` file containing the private key in this directory
mkdir -p ~/.config/sops/age
age-keygen -o ~/.config/sops/age/keys.txt
```

This creates `keys.txt`, which contains your private key, along with a comment displaying the corresponding public key.
