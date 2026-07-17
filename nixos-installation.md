A nixos installation can be divided into the following stages:

1. Partition the disk
2. Generate hardware configuration
3. Update the repository
4. Install NixOS

## Partition the disk


## firmware

Software stored in the motherboard itself. You make the choice when you buy the hardware.

Two options; UEFI and BIOS (`legacy`).
``
If `/sys/firmware/efi` exists, the firmware is UEFI, otherwise it is BIOS.

```sh
ls /sys/firmware/efi
```
## Partitions

Partition table is data at the start (sometimes also at the end) of a disk describing partitions (their locations, sizes, types, etc.).

Two options; GPT or MBR.

  - UEFI → GPT
    UEFI looks for an EFI System Partition (`ESP`) to boot. This partition type is supported by the GUID Partition Table (GPT).

  - BIOS → MBR
    Legacy BIOS boots by looking at the very first sectors of the disk, called the Master Boot Record (MBR). Can also boot from a disk using GPT with the right setup.

Partitions are virtual divisions on a single physical disk appearing as multiple independent drives. The number and type of partitions depend on:

  - Motherboard firmware
  - Filesystems to be used
  - Whether encryption is required
  - Whether a swap partition is required


## Filesystem

Internal structure used by the partition to write/retrieve data.

The filesystem is created in the actual partition — the tags you may see in the partition table are only hints to the operating system (ignored by Linux).

- [[ext4]]
  A traditional "modify-in-place" filesystem that requires very little computational power. It writes data to disk and forgets about it.

- Btrfs
  Every single read and write operation requires Btrfs to calculate and verify cryptographic checksums, manage metadata pointers for Copy-on-Write (CoW), and handle background compression (if enabled). On a high-end desktop, you won't notice this. On a laptop running on battery, or older hardware, Btrfs will consume noticeably more CPU cycles and RAM than ext4.

- ZFS
  Good for dedicated storage server (NAS), hypervisor, or any multi-drive array requiring RAID-5/6 parity where absolute data integrity and proven enterprise maturity are non-negotiable.


## Wipe data

Creating a new partition table and filesystems does not override previous data, which sometimes can cause errors (the new partition table did not override all sectors used by the previous one or one of the new partitions may be using the same sectors of a previous partition).

When reusing a disk always do one of the following:

  - Override all previous data with random data (`dd if=/dev/urandom of=/dev/sdX bs=4M status=progress`) — most secure and reliable but very slow, overkill for personal use devices.

  - Delete each partition filesystem (`wipefs -a /dev/sdXY`) and then the partition table (`wipefs -a /dev/sdX`) itself.

To verify, first inform the OS about the changes (`partprove /dev/sdX`) and then review with `lsblk -l /dev/sdX`.

