# Sops

[SOPS](https://github.com/getsops/sops) (Secrets Operations) is an editor of encrypted files that supports YAML, JSON, ENV, INI and BINARY formats and encrypts with AWS KMS, GCP KMS, Azure Key Vault, HuaweiCloud KMS, age, and PGP.

SOPS makes working with secret files seamless — instead of decrypting the file first, then edit/save it to then re-encrypt it back again, all you need to do is `sops file.yaml` and SOPS takes care of the rest.

Additionally, you don't need to deal with all those cryptic public keys each time you want to encrypt a file, SOPS automates that too with rules.


### Asymetric encryption

SOPS uses public/private keys for encryption/decryption — its documentation [recommends](https://getsops.io/docs/usage/identities/age/) using [[age]].

If you don't yet have a key pair, create one with the following command:

```sh
mkdir -p ~/.config/sops/age
age-keygen -o ~/.config/sops/age/keys.txt
```

SOPS looks for your age private key file in the following locations:

  1. `SOPS_AGE_KEY_FILE` — path to key file.
  2. `~/.config/sops/age/keys.txt`  — default path.

Or you can set the content of the key directly in the `SOPS_AGE_KEY` environment variable.


### Encrypt

Create a `.sops.yaml` file at the root of the project — SOPS searches for this file starting from your current working directory and moving upwards.

Its main purpose is to automate and standardize how secrets are encrypted by defining which encryption keys to use for different files, removing the need to specify them manually each time.

```yaml
# .sops.yaml
keys:
  # Crate aliases for your keys (easier to read and avoids repetition)
  - &alice age1devpublickey...
  - &john age1prodpublickey...
  - &admin age1prodpublickey...

creation_rules:
  # SOPS will processes the rules sequentially, from top to bottom, and
  # apply the first rule whose `path_regex` matches the file being encrypted.
  - path_regex: /users/alice/secrets.yaml$
    age: *alice
  - path_regex: .*\.prod\.yaml$
    age: *alice, *john # use multiple keys
  - age: *admin # catches all other files
```

Now you can use `sops file.yaml` to create an encrypted file or `sops -e file.yaml` to encrypt an existing file, SOPS will automatically encrypt the file based on these rules.

Notice how you can list multiple keys (e.g., age and pgp) in a single rule — SOPS encrypts the data key with all of them, allowing anyone with any of the corresponding private keys to decrypt the file.

You can skip `.sops.yaml` rules by explicitly passing a public key for encryption:

```sh
sops -e --age age1abc... myfile.txt
```


#### The role of the data key

When sops encrypts a file, it doesn't encrypt the entire file with your public key. Instead, it generates a unique data key for that file. This data key is what actually encrypts the secrets. Your public key is then used to encrypt this data key, and the encrypted data key is stored in the file's header.

When you add a new public key to `.sops.yaml` you can execute `sops updatekeys file.yaml` to update the file's header to reflect the new list of public keys from `.sops.yaml`. However, the actual data key (which is used to decrypt the secrets) is not changed.

When you remove a public key from `sops.yaml` you must execute `sops updatekeys file.yaml` and then `sops rotate -i file.yaml`. This generates a brand new data key, encrypts all of the file's secrets again, and then encrypts this new data key with the updated list of recipients.


### Decrypt

```sh
sops -d encrypted-file.yaml`
```

When you decrypt a file, SOPS:

1. Reads the encrypted file's metadata — each SOPS-encrypted file contains a `.sops` section at the top that lists which master keys were used to encrypt it.
2. SOPS looks for the corresponding private keys in your local environment (`~/.config/sops/age/keys.txt` or `SOPS_AGE_KEY`).
3. Attempts decryption with each available key — if you have any of the private keys that match the encryption recipients, decryption succeeds.


### Editing

```sh
sops encrypted-file.txt
```

This command uses your default `$EDITOR` to open a temporary file containing the decrypted content.

SOPS will re-encrypt the file using the same public keys that were already in its header. It does not re-evaluate the `.sops.yaml` file to determine who the new recipients should be.

