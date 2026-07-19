# Virtual File System (VFS)

#TODO 

> [!info]
> Also known as Linux Directory Tree, System Root or Filesystem Hierarchy Standard (FHS).

The [[disk-filesystem]] (on-disk structure) is the block-level layout (like ext4, Btrfs, or Zfs) formatted onto a disk/partition. It contains the actual data blocks and inodes.

The Target Tree (In-memory representation): The VFS is a kernel abstraction. It creates a single, unified tree structure out of all your separate storage devices.

The Mount Point: Any directory within that VFS tree (like /mnt, /media, or /home) that acts as a gateway. When you hook an on-disk filesystem to a directory, the VFS seamlessly stitches that partition's contents into the main tree.

## Device node vs. Block device

A _device node_ (also called a device file) is a special type of file that lives in your virtual file system — almost always in the `/dev/` directory (e.g. `/dev/sda` or `/dev/nvme0n1`). It doesn't contain data, instead it acts as a portal or an access point. When a program opens a device node, the Linux kernel redirects those read/write calls directly to a specific hardware driver.

A _block device_ is the actual hardware or storage abstraction. Specifically, a block device in Linux is a category of hardware that handles data in fixed-size blocks (usually 4096 bytes) and allows random access (you can jump to any part of the disk instantly). Examples include SSDs, HDDs, NVMe drives, and loop devices.

So, when you look at `/dev/sda`, you are looking at a device node that points to a block device .

> [!info]
> The reason these two aren't synonyms is that device nodes can represent things other than block devices.
