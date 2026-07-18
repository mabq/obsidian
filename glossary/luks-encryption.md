# Luks encryption

LUKS (Linux Unified Key Setup) is the standard specification for hard disk encryption on Linux. 

Instead of being a standalone program, it defines a standard on-disk format that integrates seamlessly with Linux tools to encrypt entire block devices — like partitions, full hard drives, or USB sticks.


### How to encrypting a block device

> [!warning]
> This process deletes any current filesystem in the block device. Always double-check the device node with `lsblk` before pressing enter!

First, initialize the LUKS partition. This steps creates the LUKS header and sets up your passphrase. It uses LUKS2 by default on modern systems.

```sh
sudo cryptsetup luksFormat /dev/sdXN
# Type YES in all caps to confirm, then enter your passphrase
```

> [!info]
> You can change the passphare afterwards if you need to.

Open the encrypted device. The block device remains encrypted, so Linux creates a virtual blocked device (`/dev/mapper/<NAME>`) for you to interact with — read [[#The Two Distinct Layers]] below.

```sh
# Replace `<NAME>` with whatever name you want to give to the virtual unlocked device
sudo cryptsetup open /dev/sdXN <NAME>
```

Now that the device is unlocked and mapped, format the virtual unlocked device (not the raw partition) with a filesystem.
	
```sh
# Use whatever files system you want
sudo mkfs.<FILESYSTEM> /dev/mapper/<NAME>
```

Finally, create a mount point and mount the virtual unlocked device just like a regular drive.

```sh
sudo mkdir -p /mnt/secure
sudo mount /dev/mapper/<NAME> /mnt/secure
```

Done!

### How to safely close the drive

When you are done using the drive, you need to unmount the filesystem and lock the LUKS container to secure the data again.

```sh
# Unmount the filesystem
sudo umount /mnt/secure

# Lock the LUKS container
sudo cryptsetup close <NAME>
```

Now the virtual device under `/dev/mapper/<NAME>` vanishes, and the data is completely locked away until the next time you run `cryptsetup open`.

---

### The Two Distinct Layers

When you open an encrypted LUKS partition, you are interacting with two completely different representations of the same storage space: the raw, locked hardware and the virtual, unlocked interface.

Because they do two entirely different jobs, Linux gives them two separate device nodes.










