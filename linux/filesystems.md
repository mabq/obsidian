# Filesystems

A filesystem decides how data is organized, named, stored, and retrieved on a storage device. Without one, a disk is just a flat sequence of bytes — the filesystem is what turns that into files, directories, permissions, and metadata like timestamps.

### Types of Filesystems

There isn't one universal filesystem because different storage media, workloads, and environments have different needs. Broadly, filesystems fall into a few categories:

#### Disk-based (local) filesystems

These store data persistently on physical or virtual block devices (HDDs, SSDs, USB drives).

- **ext4** — The long-standing default on most Linux distributions. Mature, reliable, journaled (meaning it logs changes before committing them, so a crash mid-write doesn't corrupt the whole disk). Less features than modern alternatives.
- **Btrfs** — A modern "copy-on-write" filesystem supporting snapshots, compression, subvolumes, built-in RAID-like redundancy, and checksumming to detect silent data corruption. All these features require resources.
- **XFS** — Designed for high performance with large files and parallel I/O; common on servers handling big datasets.
- **NTFS** — Windows' native filesystem; Linux can read/write it for interoperability.
- **FAT32 / exFAT** — Simple, ubiquitous formats with minimal metadata overhead, used for USB drives and SD cards specifically because nearly every operating system (Windows, macOS, Linux, cameras, game consoles) can read them.
- **APFS** — Apple's filesystem, optimized for SSDs and encryption.

These exist because there are real trade-offs between reliability, performance, feature richness (snapshots, compression, encryption), and compatibility. A journaling filesystem like ext4 costs a bit of write performance in exchange for crash safety; FAT32 sacrifices modern features for near-universal compatibility.

#### Network filesystems

These let a machine access files that physically live on another machine over a network, as if they were local.

- **NFS (Network File System)** — Common in Unix/Linux environments.
- **SMB/CIFS** — Common in Windows environments (and used by Samba on Linux to interoperate).

They exist to let multiple machines share a common pool of files without copying them around manually.

#### Pseudo (virtual) filesystems

These don't store data on a disk at all — they present kernel or process information *as if* it were a filesystem, because the file/directory interface is a convenient, universal way to expose data.

- **procfs (`/proc`)** — Exposes running process and kernel information as virtual files.
- **sysfs (`/sys`)** — Exposes device and driver information.
- **tmpfs** — Stores files in RAM instead of on disk, for speed and automatic cleanup (used for `/tmp` on many systems).
- **devtmpfs** — Automatically provides device nodes in `/dev`.

They exist because representing dynamic, in-memory kernel state as "files you can `cat`" is simpler than inventing a separate API for every subsystem.

#### Special-purpose filesystems

- **squashfs** — A read-only, compressed filesystem, often used for live CDs, installer images, or embedded systems where the contents never change.
- **overlayfs** — A "union" filesystem that layers one filesystem on top of another (a lower read-only layer and an upper writable layer). This is the technology that makes Docker container images work efficiently.
- **cgroup filesystem** — Exposes Linux control groups (resource limits) as a filesystem.

The common thread across all of these: a filesystem's *design* follows from the problem it's solving — durability on spinning disks, speed on RAM, cross-platform sharing, exposing live kernel state, or layering read-only and writable data.


### The Linux VFS (Virtual Filesystem Switch/Layer)

^64c009

Given how many different filesystem types exist, how does a Linux system let you `cd`, `ls`, and `cat` the same way regardless of whether a directory lives on an ext4 disk, an NFS share, or a tmpfs RAM-backed mount?

This is the job of the **VFS — the Virtual File System**, an abstraction layer inside the Linux kernel.

**What the VFS does:**
- It defines a common set of operations — `open()`, `read()`, `write()`, `close()`, `mkdir()`, `rename()`, and so on — that every filesystem driver must implement.
- Application programs and system utilities only ever talk to the VFS. They never talk to ext4 or NFS or tmpfs directly.
- The VFS translates those generic calls into filesystem-specific operations behind the scenes, using the driver registered for that particular filesystem type.

**Core data structures it uses to do this:**
- **Superblock** — represents an entire mounted filesystem instance (its size, block size, state).
- **Inode** — represents a single file or directory's metadata (permissions, owner, timestamps, size, and pointers to where the actual data blocks are) — separate from the file's name.
- **Dentry (directory entry)** — maps a name to an inode, and is what makes path lookups (like `/home/user/file.txt`) fast, since dentries are cached.
- **File object** — represents an open file from a specific process's point of view (its current read/write offset, for example).

Because every filesystem driver conforms to this same interface and populates these same structures, the kernel — and every program running on it — can treat wildly different underlying storage systems identically. This is also what makes it possible to write a new filesystem driver without having to change every application that will use it.

```
+-------------------------------------------------------------+
|               User Applications (e.g., Neovim)              |
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


### How These Filesystems Coexist Within a Single Root Filesystem

Unlike Windows, which assigns each filesystem its own drive letter (`C:`, `D:`, `E:`), Linux presents everything as **one single, unified directory tree**, starting at `/` (the root).

Here's how that works:

- One filesystem is mounted at `/` itself — this is the **root filesystem**, typically ext4, XFS, or Btrfs, containing the core OS.
- Every other filesystem is attached ("mounted") onto an existing directory within that tree, called a **mount point**. For example:
  - `/home` might be its own ext4 partition, mounted separately from root (common so user data survives an OS reinstall).
  - `/boot` might be a small dedicated partition for the bootloader.
  - `/mnt/usb-drive` might be a FAT32-formatted USB stick.
  - `/proc` and `/sys` are the pseudo-filesystems described above, mounted automatically by the kernel at boot.
  - `/tmp` might be tmpfs, living in RAM.
  - A remote NFS share might be mounted at `/mnt/shared-data`.

When you navigate into `/home`, the VFS transparently switches which underlying filesystem driver is handling your requests — you'd never know from the `ls` output alone that you just crossed from one filesystem into a completely different one.

#### How to Recognize Which Filesystem Is Mounted Where

A few standard tools let you inspect this:

- **findmnt** — lists all mounted filesystems, showing the mount point, source, filesystem type and mount options.
- **`df -Tf`** — shows disk usage per mounted filesystem, including the `-T` flag for filesystem type.
- **`lsblk -f`** — lists block devices (disks/partitions) along with the filesystem type and label on each.
- **`cat /proc/mounts`** or **`cat /proc/self/mounts`** — the live, kernel-reported list of all mounts (what `mount` itself reads from).
- **`cat /etc/fstab`** — the *configuration* file listing filesystems that should be mounted automatically at boot (not necessarily what's mounted right now).
- **`blkid`** — reports the filesystem type and UUID for each block device, useful for identifying a disk/partition before it's even mounted.
- **`stat -f /some/path`** — reports filesystem-level information (type, block size, free space) for whatever filesystem a given path lives on.
- **`cat /proc/filesystems`** — lists every filesystem type the currently running kernel has support for (compiled in or loaded as a module), whether or not it's currently in use.

#### Putting It Together

In practice, a running Linux system might simultaneously have:

- a disk-filesystem mounted at `/`
- a separate disk-filesystem partition mounted at `/home`
- tmpfs mounted at `/tmp` and `/run`
- procfs mounted at `/proc`
- sysfs mounted at `/sys`
- overlayfs mounted somewhere under `/var/lib/docker` if containers are running
- an NFS share mounted at `/mnt/nas`

All of it appears as one seamless tree to the user, all of it is accessed through the same system calls, and all of it is made possible by the VFS quietly routing each request to the correct underlying filesystem driver.


### FAQs

#### What is a Filesystem Hierarchy Standard (FHS)

^59afc8

A specification describing where files should be stored on Unix/Linux systems — like many other standarization specs, it is managed by [freedesktop.org](https://specifications.freedesktop.org/fhs/latest/).

|Directory|Purpose|
|-|-|
|`/`|Root of the entire filesystem hierarchy|
|`/bin`|Essential command binaries for all users (e.g., `ls`, `cp`, `cat`)|
|`/boot`|Static files for booting (kernel, initramfs, bootloader)|
|`/dev`|Device files (e.g., `/dev/null`, disk devices)|
|`/etc`|Host-specific configuration files (no binaries)|
|`/home`|User home directories (not always strictly required by FHS but universally used)|
|`/lib`|Essential shared libraries and kernel modules|
|`/mnt`|Temporary mount points for manual filesystem mounting|
|`/proc`|Process information
|`/root`|Home directory for the root user|
|`/run`|Runtime variable data (cleared on reboot, e.g., PID files)|
|`/srv`|Site-specific data (e.g., web server files)|
|`/sys`|Kernel information
|`/tmp`|Temporary files|
|`/usr`|Secondary hierarchy for user utilities, libraries, and applications (shareable, mostly read-only). Subdirs include `/usr/bin`, `/usr/lib`, `/usr/share`, etc.|
|`/var`|Variable data (logs, spools, caches). Subdirs: `/var/log`, `/var/spool`, `/var/cache`, etc.|

Nix completely rejects the FHS because it assumes a global, shared environment where all applications share the same folders for binaries, libraries and configurations, making reproduceability impossible. [[nix#^697838|More info]]


#### The "chicken-and-egg" paradox

If `/dev/sdXN` is a file path inside `/`, how can `/dev/sdXN` contain the filesystem that is mounted into `/`?

The answer to this lies in the bootstrapping sequence, on a typical Linux boot, there are three distinct stages:

1. **Kernel only:** — detects hardware and sets up essential internal structures.
2. **Temporary root (initramfs in RAM):** — contains a minimal `/`, including `/dev`, and prepares the real storage.
3. **Real root:** — the filesystem from `/dev/sdXY` becomes `/`, and pseudo/virtual filesystems are mounted into `/dev`, `/proc`, and `/sys` (empty directories in `/dev/sdXY` that now show the pseudo/virtual filesystems created by the Kernel in memory).

> [!info]
> Mounting a fileystem into a directory shadows the contents of that directory — try mounting the filesystem of a external drive into `~/Downloads` and then unmounting it.


#### Device node vs. Block device

A device node (also called a device file) is a special type of file that lives in your virtual file system — almost always in the `/dev/` directory (e.g. `/dev/sda` or `/dev/nvme0n1`). It doesn't contain data, instead it acts as a portal or an access point. When a program opens a device node, the Linux kernel redirects those read/write calls directly to a specific hardware driver.

A block device is the actual hardware or storage abstraction. Specifically, a block device in Linux is a category of hardware that handles data in fixed-size blocks (usually 4096 bytes) and allows random access (you can jump to any part of the disk instantly). Examples include SSDs, HDDs, NVMe drives, and loop devices.

So, when you look at `/dev/sdXN`, you are looking at a device node that points to a block device .

The reason these two aren't synonyms is that a device node can represent things other than block devices.


#### Mount exFAT filesystems

 exFAT has no on-disk concept of Unix ownership or permission bits, so the kernel driver fabricates a single, uniform set of permissions for the entire filesystem at mount time, based on options you pass in. By default many distros mount it with a restrictive `umask` and `uid`/`gid` pinned to whoever mounted it, which is why it looks locked to just you.
 
To open it up to other users (like plex), you control this with mount options:

```sh
sudo mount -t exfat -o uid=$(id -u),gid=$(id -g),umask=0022 /dev/sdXY /mnt/<MOUNT_POINT>
```
