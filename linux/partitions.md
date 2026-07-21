# Partitions

Virtual divisions on a single physical disk appearing as multiple independent drives. 

### Partition table

When a disk is partitioned, a small amount of space at the very beginning (or end) of the drive is reserved for a partition table — a data structure containing information about each partition (location, size, type).

>[!tip]
> Always create a partition table. Disks with no partition table might be flagged as "uninitialized" or empty by operating systems, causing confusion that may lead a user to format the disk, destroying its data.
>
> Creating a [[filesystems|filesystem]] directly on a raw unpartitioned disk is possible, but only recommended for disks aimed to be used in storage pools (ZFS / Btrfs / LVM) or in cloud or VM environments.

The two common standards are:

- **GPT** (GUID Partition Table)
	The modern standard and default choice for UEFI systems.
	Supports 128 primary partitions and disks up to 9.4 Zettabytes.
<br>	
- **MBR** (Master Boot Record)
	The legacy fallback and default choice for BIOS systems.
	Supports 4 primary partitions and disks up to 2 Terabytes.

See [[firmware]] for more information about UEFI and BIOS.

### Partition types

Simple hex-codes marking the purpose of each partition in the partition table — used by programs to quickly find a partition without reading its content.

> [!info]
> The same way you can fill a jar labeled "sugar" with salt, you can fill a partition other content than the one marked on the partition table.

> [!info]
> GPT and MBR use different hex-codes for the same partition types — partition tools normally abstract this.

The most common partition types in Linux are:

- **ESP (EFI System Partition)**
	Marks the partition containing the boot loader used by UEFI systems.
	The official UEFI specification requires the partition to be formatted with a `FAT32` filesystem.
<br>	
- **BIOS Boot Partition**
	Marks the partition used by BIOS firmware only when booting from a drive with a GPT partition table. Bios firmware requires the partition to remain unformatted (no filesystem).
	GRUB uses this raw unformatted space to insert its core stage-2 boot-code.
<br>	
- **Linux root**
	Marks the partition containing the operating system.
	Typically formatted with `ext4` or `btrfs` filesystems.
	Used by systemd to automatically mount the filesystem in that partition to `/` without relying on an `/etc/fstab` file.
	Can be wrapped in a LUKS layer for encryption.
<br>	
- **Linux home**
	Marks the partition containing user data and personal configurations.
	Typically formatted with `ext4` or `btrfs` filesystems.
	Used by systemd to automatically mount the filesystem in that partition to `/home`.
	Can be wrapped in a LUKS layer for encryption.
<br>	
- **Linux swap**
	Marks the partition to be used by the kernel as swap when physical memory capacity is exceeded. Must remain unformatted (no filesystem), swap uses its own raw, unique block formatting structured exclusively for page swapping.
	Should be encrypted to avoid writing sensitive data (like passwords, open documents, or encryption keys that were sitting in memory) directly onto the raw disk in plain text.
<br>	
- **Linux LVM (Logical Volume Manager)**
	Marks the partition acting as an LVM "Physical Volume".
	The partition must remain unformatted, filesystems are created inside the logical volumes.
	
> [!tip]
> Use `lsblk -o NAME,FSTYPE,PARTTYPE` to show the filesystem and partition type code side to side. 


### Disk layout

How a disk is partitioned depends on many factors.

- **Motherboard firmware**
  For UEFI always use a GPT partition table with a ESP partition. For BIOS, only use GPT (with a BIOS boot partition) when the boot drive is greater than 2TB or when more than 4 primary partitions are required, in any other case prefer MBR (no boot partition required).
<br>
- **Filesystem**
  Modern [[filesystems|filesystems]] like `btrfs` can replace the need for partitions, dynamic-sizing subvolumes can coexist inside a single partition. With older filesystems like `ext4` you do need to partition your disk (guessing the appropieate size for each partition ahead of time).
<br>
- **Swap partitition**
  Not recommended anymore, swap files perform just as good and can be removed or resized when needed.


### FAQs

#### How to create disk partitions?

Read `man parted`.
  
#### How to encrypt a partition?

See [[luks| LUKS encryption]].

#### How to delete a filesystem?

Deleting the filesystem makes the block device appear empty — data is deleted for practical terms (it can still be recovered with specialized tools).

```sh
# ⚠️ Double check device node before pressing Enter!
wipefs -a /dev/sdX[0-9]*
```

#### How to delete a partition table?

Deleting the partition table makes the physical disk appear empty — data is deleted for practical terms (it can still be recovered with specialized tools).

```sh
# ⚠️ Double check device node before pressing Enter!
wipefs -a /dev/sdX
```

#### How to actually delete data?

 ⚠️ Executing theses commands will make data **unrecoverable**.

| Scenario                   | Recommended Tool/Command                            | Notes                               |
| -------------------------- | --------------------------------------------------- | ----------------------------------- |
| Single file                | `shred -u -v -z file`                               | Best & easiest                      |
| Entire HDD                 | `shred -v -n 3 -z /dev/sdX` or ATA Secure Erase     | Multiple passes for paranoia        |
| SSD / NVMe                 | Use `nvme-cli` or `hdparm` Secure Erase             | Software overwrites are ineffective |
| Quick wipe (non-sensitive) | `dd if=/dev/zero of=/dev/sdX bs=4M status=progress` | Fast zeroing                        |
| Maximum security           | `shred` with 7+ passes + final zero                 | Overkill for most                   |
