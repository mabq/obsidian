# age

Modern replacement for GnuPG — avoids complex subkey configurations, outdated defaults, and confusing flags in favor of a clean, misuse-resistant interface.

Usage:

```sh
# Encrypt
age --passphrase -o <output-file>.age <input-file>

# Decrypt to file
age --decrypt -o <output-file> <input-file>.age

# Decrypt to stdout
age --decrypt <input-file>.age
```
