# Linux Kernel

The abstraction layer between hardware and software.


### Turning Hardware into Data Structures

At the lowest level, your CPU, RAM, and peripherals only understand raw voltages, binary machine code, and hardware interrupts. The kernel's job is to abstract this chaotic physical reality into clean, manageable data structures inside the system memory.

- **Hardware Interrupts to Task Lists**
  When you press a key or a packet hits your network card, the hardware sends a raw electrical signal (an interrupt). The kernel catches this and translates it into an entry in a software queue or a task list (`task_struct`), scheduling it just like any other piece of data.
<br>
- **Silicon to Filesystems**
  A hard drive or SSD is just a massive block of raw, numbered sectors holding binary data. The kernel uses data structures (like **inodes**, **superblocks**, and **dentry cache** objects) to present that raw block layout to you as a beautiful, nested tree of files and directories.  ^150a1b


### Turning Data Structures into Hardware Actions

When you write a program, you don't think about CPU registers or memory voltage. You think in data structures: arrays, strings, objects, and file descriptors.

When your code wants to do something real, it talks to the kernel using **system calls** (syscalls). The kernel takes your high-level data structures and converts them back down into raw machine actions:

```
[ Your Program ]
  └─ Uses high-level data structures (e.g., an array of strings in memory)
       │
  [ System Call (e.g., sys_write) ]  <-- The Transition Zone
       │
[ Linux Kernel ]
  └─ Translates structures into page tables, buffer allocations, and driver tasks
       │
[ Hardware / CPU / Storage ]
  └─ Executes raw binary instructions and manipulates physical voltages

```


### The Ultimate Illusion: Virtual Memory

The best example of the kernel acting as this specific middleman is **Virtual Memory**.

To your program, memory looks like a single, massive, contiguous array of bytes (a perfectly clean data structure). In reality, the physical RAM chips might be heavily fragmented, or parts of your program might have been kicked out to a swap partition on disk.

The kernel sits in the middle, using a data structure called a **Page Table**. Every time a raw binary instruction tries to read a memory address, the kernel and the CPU's Memory Management Unit (MMU) instantly map that logical structure to the actual physical silicon address.