# Block device & Device node


### The Device Node (The Interface)

A device node (also called a device file) is a special type of file that lives in your [[virtual-file-system|filesystem]] — almost always in the `/dev/` directory (e.g. `/dev/sda` or `/dev/nvme0n1`).

It doesn't contain data on your hard drive. Instead, it acts as a portal or an access point.

When a program opens a device node, the Linux kernel redirects those read/write calls directly to a specific hardware driver.


### The Block Device (The Subtype)

A block device is the actual hardware or storage abstraction.

Specifically, a block device in Linux is a category of hardware that handles data in fixed-size blocks (usually 4096 bytes) and allows random access (you can jump to any part of the disk instantly). Examples include SSDs, HDDs, NVMe drives, and loop devices.

> [!tldr]
>
> A _block device_ is the concept/hardware. A device node is the file type interface. When you look at `/dev/sda`, you are looking at a device node that points to a _block device_.
>
> The reason these two aren't synonyms is that device nodes can represent things other than block devices.

