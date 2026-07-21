# Linux Virtual Filesystem (VFS)

The **Virtual File System (VFS)** is an abstract software layer in the Linux kernel that sits between user-space applications and concrete storage [[filesystems|filesystems]] like Btrfs, Ext4, or XFS.

Its primary job is to provide a uniform, standardized interface (using standard system calls like `open()`, `read()`, and `write()`) so that applications don't need to know or care what filesystem or underlying hardware they are interacting with.

### Why It Matters: The Power of Abstraction

Without the VFS, if you wanted to read a file from a USB drive formatted in FAT32, an SSD using Btrfs, or a network share using NFS, your text editor would need three completely different code tracks to talk to those distinct systems.

With VFS, the kernel provides a single unified interface:

```
+-------------------------------------------------------------+
|            User Applications (e.g., Yazi, Neovim)           |
+-------------------------------------------------------------+
|      Standard System Calls (open, read, write, close)       |
+-------------------------------------------------------------+
|                   VIRTUAL FILE SYSTEM (VFS)                 | (Abstraction Layer)
+-------------------------------------------------------------+
|    Btrfs    |    Ext4     |    XFS      |    sysfs/procfs   | (Concrete Filesystems)
+-------------+-------------+-------------+-------------------+
|  SATA SSD   |  NVMe M.2   | HDD Storage |   Kernel Memory   | (Physical/Virtual Media)
+-------------------------------------------------------------+

```


### The Four Primary VFS Objects

The Linux VFS treats everything through an object-oriented paradigm implemented in C using structures. It relies on four fundamental object types:

* **Superblock** (`super_block`)
  Represents a specific mounted filesystem. It contains metadata about the filesystem as a whole, such as its block size, total size, and status flags.
<br>
* **Inode** (`inode`)
  Represents a specific physical file or directory on disk. Crucially, it contains all the metadata about the file (permissions, size, timestamps, ownership) and pointers to the actual data blocks, but it does **not** contain the file's name.
<br>
* **Dentry** (`dentry`)
  Short for "directory entry." This object represents a specific component in a file path (e.g., in `/home/user/notes`; `/`, `home`, `user`, and `notes` are all dentries). Dentries link file names to their corresponding inodes and are cached heavily in RAM for rapid path lookups.
<br>
* **File** (`file`)
  Represents an *open* file associated with a specific process. It tracks process-specific states like the current file offset (where the read/write cursor is) and the access mode (read-only, write-only).

See [[linux-kernel#^150a1b|Silicon to filesystems]].

### Everything is a File

Because the VFS abstracts everything into these standard structures, it allows Linux to treat completely different concepts as files. Beyond local storage disks, VFS hooks into:

- **Network storage:** Things like NFS, SMB.
- **Pseudo-filesystems:** `procfs` (`/proc`) and `sysfs` (`/sys`), which don't exist on disk at all but are exposed by the VFS as a way to read and write directly to kernel memory and hardware state using text files.
- **Devices:** Hardware devices mapped under `/dev` (like `/dev/sda` or audio channels).

This seamless abstraction is precisely what enables command-line pipelines like `cat /proc/cpuinfo | grep "model name"` to work identically to reading a plain text file from your home directory.


### The "chicken-and-egg" paradox

If `/dev/sdXN` is a file path inside the `/` directory, how can `/dev/sdXN` have the filesystem that is mounted into `/`?

The answer to this paradox lies in the bootstrapping sequence,  on a typical Linux boot, there are three distinct stages:

1. **Kernel only:** detects hardware and sets up essential internal structures.
2. **Temporary root (initramfs in RAM):** contains a minimal `/`, including `/dev`, and prepares the real storage.
3. **Real root:** the filesystem from your SSD (for example, on `/dev/sdXY`) becomes `/`, and the **virtual filesystems** such as `/dev`, `/proc`, and `/sys` are mounted into it — the contents of these directories come from memory, not the SSD.

This layered startup process is what avoids the apparent "chicken-and-egg" problem. The kernel never needs to find `/dev/sdXN` by reading it from the disk; it creates the device node from its knowledge of the detected hardware, then uses it to access the filesystem stored on the partition.

> [!info]
> Any filesystem can be mounted into any directory in the VFS — try mounting the filesystem of a external disk into `~/Downloads`.

### Device node vs. Block device

A **device node** (also called a device file) is a special type of file that lives in your virtual file system — almost always in the `/dev/` directory (e.g. `/dev/sda` or `/dev/nvme0n1`). It doesn't contain data, instead it acts as a portal or an access point. When a program opens a device node, the Linux kernel redirects those read/write calls directly to a specific hardware driver.

A **block device** is the actual hardware or storage abstraction. Specifically, a block device in Linux is a category of hardware that handles data in fixed-size blocks (usually 4096 bytes) and allows random access (you can jump to any part of the disk instantly). Examples include SSDs, HDDs, NVMe drives, and loop devices.

So, when you look at `/dev/sdXN`, you are looking at a device node that points to a block device .

The reason these two aren't synonyms is that a device node can represent things other than block devices.
