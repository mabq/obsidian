# Disk partition

Partitioning a disk allows you to have virtual divisions on a single physical disk appearing as multiple independent drives.

Each partition has its own filesystem, which can be mounted

Partitioning a disk allows you to have virtual divisions on a single physical disk appearing as multiple independent drives. 

## Partition table

Data structure on a storage device (like a hard drive or SSD) that defines how the disk is divided into partitions, including their locations, sizes, and types.

Two options:

- GPT (GUID Partition Table)
	Default choice for UEFI systems.
	Supports 128 primary partitions and disks up to 9.4 Zettabytes.
		
- MBR
	Default choice for BIOS systems (legacy).
	Supports only 4 primary partitions and disks up to 2 Terabytes.

> [!info]
> Use GPT on BIOS only if you want to attach a large modern storage drive (greater than 2 TB) as your boot drive, or you need a complex partitioning layout with lots of native partitions.


## Partitions


Each partition has its own [[filesystem]].

## Actions

#### Delete data fast (not securely)

If _you_ (not someone else!) plan to reuse a disk for a fresh installation or a new LVM/RAID setup, execute:

> [!warning]
> This will destroy the pointers to the data, recovering it will require specialized tools.

```sh
# Wipes filesystem signatures from all partitions
wipefs -a /dev/sdX[0-9]*

# Wipe the partition table itself (disk-level)
wipefs -a /dev/sdX
```

#### Delete data securely

If you are disposing of the drive or giving it to someone else and want to ensure your actual data cannot be recovered, execute:

> [!warning]
> This will completely destroy the data, recovering it will not be possible!

```sh
# Always double-check the device node before hitting Enter!
dd if=/dev/urandom of=/dev/sdX bs=4M status=progress
```
