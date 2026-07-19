# Luks encryption

LUKS (Linux Unified Key Setup) defines a standard on-disk format that integrates seamlessly with Linux tools to encrypt entire block devices — like partitions, full hard drives, or USB sticks.

> [!info]
> In examples below, replace `/dev/sdXN` with a proper device node.

### Encrypt a block device

All you need to do to encrypt a block device is to format it with LUKS — this steps creates the LUKS header and sets up your passphrase. It uses LUKS2 by default on modern systems.

```sh
# ⚠️ Double check the device node before pressing Enter!
sudo cryptsetup luksFormat /dev/sdXN
```

When prompted, type "YES" in all caps to confirm and then enter your decryption passphrase (can be changed later).

One cannot directly interact with a encrypted block device, you need to open it first.

### Open an encrypted block device

Use the following command to open the encrypted block device — enter your decryption passphrase when prompted.

```sh
# Replace <NAME> with whatever name you want
sudo cryptsetup open /dev/sdXN <NAME>
```

When you open a encrypted block device, the Linux kernel's device-mapper driver (`dm-crypt`) creates a virtual, unlocked block device at `/dev/mapper/<NAME>` — this unlocked block device is what you actually format with a [[filesystem|filesystem]], mount to your [[linux-vfs|vfs]] and interact with (think of it as the interface for the encrypted block device). 

### Format and mount the unlocked block device

Format the unlocked block device with whatever [[filesystem|filesystem]] you want.
	
```sh
sudo mkfs.<FILESYSTEM> /dev/mapper/<NAME>
```

Mount the unlocked block device the same way you mount any other drive.

```sh
sudo mount /dev/mapper/<NAME> /mnt/<DIR>
```

### Close the unlocked block device

When you are done using the drive, you need to unmount the filesystem and lock the LUKS container to secure the data again.

```sh
sudo umount /mnt/<DIR>
sudo cryptsetup close <NAME>
```

Now the unlocked device vanishes, and the data is completely locked away until the next time you run `cryptsetup open`.
