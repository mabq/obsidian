# Luks encryption

LUKS (Linux Unified Key Setup) defines a standard on-disk format that integrates seamlessly with Linux tools to encrypt entire block devices — like partitions, full hard drives, or USB sticks.

## How to encrypting a block device

> [!danger]
> These commands delete data.

> [!info]
> In the example commands below, replace `/dev/sdXN` with the device node of the block device you want to encrypt (see [[virtual-file-system|virtual file system]]).
>
> Always double check the device node with `lsblk` before pressing enter.

First, apply the LUKS format to the block device. This steps creates the LUKS header and sets up your passphrase (can be changed if needed). It uses LUKS2 by default on modern systems.

```sh
# Type YES in all caps to confirm and enter your passphrase when prompted
sudo cryptsetup luksFormat /dev/sdXN
```

Then, open the encrypted block device — replace `<NAME>` with whatever name you want.

```sh
sudo cryptsetup open /dev/sdXN <NAME>
```

>[!info]
>When you open the encrypted block device, the Linux kernel's device-mapper driver (`dm-crypt`) creates a virtual, unlocked block device at `/dev/mapper/<NAME>`. When you read from or write to this block device, the kernel decrypts/encrypts the data on the fly transparently.
>
>This virtual, unlocked block device is what you actually format with a filesystem and mount to your system. 

Now, format the virtual, unlocked block device — use whatever filesystem you want.
	
```sh
sudo mkfs.<FILESYSTEM> /dev/mapper/<NAME>
```

Finally, create a mount point and mount the virtual, unlocked block device just like a regular drive.

```sh
sudo mkdir -p /mnt/secure
sudo mount /dev/mapper/<NAME> /mnt/secure
```

Done!


## Safely close the virtual, unlocked block device

When you are done using the drive, you need to unmount the filesystem and lock the LUKS container to secure the data again.

```sh
# Unmount the filesystem
sudo umount /mnt/secure

# Lock the LUKS container
sudo cryptsetup close <NAME>
```

Now the virtual, unlocked device under `/dev/mapper/<NAME>` vanishes, and the data is completely locked away until the next time you run `cryptsetup open`.
