# Disk partitions

Virtual divisions on a single physical disk appearing as multiple independent drives. 

### Partition table

To create partitions, a small amount of space at the very beginning (or end) of the drive is reserved for a partition table — a data structure containing the location, size, type, etc. about each partition.

>[!tip]
> Always create a partition table!
>
> Disks with no partition table might be flagged as "uninitialized" or empty by operating systems, causing confusion that may lead a user to format the disk, destroying its data.
>
> Creating a [[disk-filesystem|filesystem]] directly on a raw unpartitioned disk is possible, but only recommended for disks aimed to be used in storage pools (ZFS / Btrfs / LVM) or in cloud or VM environments.

The two common standards are:

- GPT (GUID Partition Table)
	The modern standard and default choice for UEFI systems.
	Supports 128 primary partitions and disks up to 9.4 Zettabytes.

- MBR (Master Boot Record)
	The legacy fallback and default choice for BIOS systems.
	Supports 4 primary partitions and disks up to 2 Terabytes.

### Partition types

Hex-codes marking the purpose of each partition in the partition table — used by programs to quickly find the partition with the content they require.

> [!info]
> The same way you can fill a jar labeled "sugar" with salt, you can fill a partition with completely different content than the one specified in its partition table.
>
> The real content of a partition is the data contained in its [[disk-filesystem|filesystem]].

> [!info]
> GPT and MBR use different hex-codes for the same partition types — partition tools normally abstract this.

The most common partition types in Linux are:

> [!tip]
> Use `lsblk -o NAME,FSTYPE,PARTTYPE` to show the filesystem and partition type code side to side. 

- **ESP (EFI System Partition)**
	Marks the partition containing the boot loader used by UEFI systems.
	The official UEFI specification requires the partition to be formatted with a `FAT32` filesystem.

- **BIOS Boot Partition**
	Marks the partition used by BIOS firmware only when booting from a drive with a GPT partition table. Bios firmware requires the partition to remain unformatted (no filesystem).
	GRUB uses this raw unformatted space to insert its core stage-2 boot-code.
	
- **Linux root**
	Marks the partition containing the operating system.
	Typically formatted with `ext4` or `btrfs` filesystems.
	Used by systemd to automatically mount the filesystem in that partition to `/` without relying on an `/etc/fstab` file.
	Can be wrapped in a LUKS layer for encryption.

- **Linux home**
	Marks the partition containing user data and personal configurations.
	Typically formatted with `ext4` or `btrfs` filesystems.
	Used by systemd to automatically mount the filesystem in that partition to `/home`.
	Can be wrapped in a LUKS layer for encryption.

- **Linux swap**
	Marks the partition to be used by the kernel as swap when physical memory capacity is exceeded. Must remain unformatted (no filesystem), swap uses its own raw, unique block formatting structured exclusively for page swapping.
	Should be encrypted to avoid writing sensitive data (like passwords, open documents, or encryption keys that were sitting in memory) directly onto the raw disk in plain text.
	
> [!info]
> Prefer swap files (easy resize, similar performance)

- **Linux LVM (Logical Volume Manager)**
	Marks the partition acting as an LVM "Physical Volume".
	The partition must remain unformatted, filesystems are created inside the logical volumes.

### Partitions layout

How a disk is partitioned depends on many factors:

**Motherboard firmware**

For UEFI always use a GPT partition table with a ESP partition.

For BIOS use a MBR parititon table (no special partition required). Only use a GPT partition table (with a BIOS boot partition) when the boot drive is greater than 2TB or when more than 4 primary partitions are required.

**Filesystem**

`btrfs` subvolumes replace the need for separate partitions — just create a single partition (which can be encrypted with LUKS) 

If you prefer `ext4` you need to assign fixed partition sizes when configuring the disk.

---

## How to...

##### Delete data fast (not securely)

If _you_ (not someone else!) plan to reuse a disk for a fresh installation or a new LVM/RAID setup, execute:

> [!warning]
> This will destroy the pointers to the data, recovering it will require specialized tools.

```sh
# Wipes filesystem signatures from all partitions
wipefs -a /dev/sdX[0-9]*

# Wipe the partition table itself (disk-level)
wipefs -a /dev/sdX
```

##### Delete data securely

If you are disposing of the drive or giving it to someone else and want to ensure your actual data cannot be recovered, execute:

> [!warning]
> This will completely destroy the data, recovering it will not be possible!

```sh
# Always double-check the device node before hitting Enter!
dd if=/dev/urandom of=/dev/sdX bs=4M status=progress
```
