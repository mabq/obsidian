# Virtual File System (VFS)

> [!info]
> Also known as _Linux Directory Tree_, _System Root_ or _Filesystem Hierarchy Standard (FHS)_.

The [[filesystem]] (on-disk structure) is the block-level layout (like ext4, Btrfs, or Zfs) formatted onto a disk/partition. It contains the actual data blocks and inodes.

The Target Tree (In-memory representation): The VFS is a kernel abstraction. It creates a single, unified tree structure out of all your separate storage devices.

The Mount Point: Any directory within that VFS tree (like /mnt, /media, or /home) that acts as a gateway. When you hook an on-disk filesystem to a directory, the VFS seamlessly stitches that partition's contents into the main tree.
