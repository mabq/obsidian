# age

A modern, minimal encryption tool that's much simpler than GPG.

## Symetric encrytion

^778d51

Use the following commands to encrypt/decrypt files using a passphrase.

> [!warning]
> Never type the passphrase inside an untrusted system — see  [[bitwarden#^990072|2FA]].

Encrypt a file (make sure you use a long, random passphrase):

```sh
# Use text-based format instead of binary encoding
age --armor --passphrase -o <entrypted-file>.age <file-to-encrypt>

# Use binary encoding
age --passphrase -o <entrypted-file>.age <file-to-encrypt>
```

Decrypt a file (make sure you don't run this comman in a directory tracked by git):

```sh
# Send decrypted content to a file
age --decrypt -o <output-file> <encrypted-file>.age

# Send decrypted content to stdout
age --decrypt <encrypted-file>.age
```


## Asymetric encryption

^09f97e

To encrypt/decrypt using a public/private key pairs.

#### Key pair

First, **create your identity** (key pair):

```bash
age-keygen -o keys.txt
```
  
This creates the file `keys.txt` containing your private and public keys. The public key is printed to the terminal and will look something like:
  
```
age1ql3z7hjy54pw3hyww5ayyfg7zqgvc7w3j2elw8zmrj2kg5sfn9aqmcac8p
```

> [!warning]
> Keep the `key.txt` file **secret and safe**. Anyone with this file can decrypt files encrypted for you.
 
#### Encrypt

Then, encrypt a file for someone, use their public key with the `-r` flag:

```bash
age -a -r <recipient1_public_key> -o secret.txt.age secret.txt
```

   - `-a` outputs text-based format instead of binary (useful for email or pasting).
   - `-r` specifies the recipient's public key.
   - `-o` specifies the output file (creates encrypted file).
   - The last argument is the file to encrypt.

Or encrypt for multiple receipients (each recipient can independently decrypt the file):

```sh
age -a -r <recipient1_public_key> -r <recipient2_public_key> -o secret.txt.age secret.txt
```

To avoid passing many public keys to the command you can create a `recipients.txt` file with one public key per line (lines starting with `#` are ignored as comments):

```
# Alice
age1ql3z7hjy54pw3hyww5ayyfg7zqgvc7w3j2elw8zmrj2kg5sfn9aqmcac8p
# Bob
age1lggyhqrw2nlhcxprm67z43rta597azn8gknawjehu9d9dl0jq3yqqvfafg
```

And then encrypt:

```bash
age -R recipients.txt file.txt > file.txt.age
```

Age can also use existing SSH keys (supported formats include ed25519 and RSA):

```bash
age -R ~/.ssh/id_ed25519.pub example.jpg > example.jpg.age
age -d -i ~/.ssh/id_ed25519 example.jpg.age > example.jpg
```
 

#### Decrypt

To decrypt a file you received, use your private key file with the `-i` (identity) flag:

```bash
age -d -i key.txt -o secret.txt secret.txt.age
```

- `-d` enables decrypt mode.
- `-i` specifies your identity file (private key).
- `-o` specifies the output file (decrypted plaintext) .

