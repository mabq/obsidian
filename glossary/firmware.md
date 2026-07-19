# Firmware

Low-level software embedded in hardware that controls basic device functions.

> [!tip]
> Buy hardware from "trusted" brands and hope for the best.

Firmware is the very first code to execute on the system, so it runs in privileged mode on the main CPU with full access to all RAM.

Closed-source firmware (almost all UEFI, BIOS, SSD, GPU, etc.) creates serious trust and supply-chain risks — companies can hide backdoors, bugs, or intentional flaws with no audit. Many experts consider it one of the biggest security weaknesses in modern PCs. For more info visit [Firmware Security Risks (Wikipedia)](https://en.wikipedia.org/wiki/Firmware#Security_risks).

### Motherboard firmware

Two options:

- UEFI (Unified Extensible Firmware Interface)
- Legacy BIOS

To identify the motherboard firmware check the directory `/sys/firmware/efi`, if it exists the firmware is UEFI, if it does not the firmware is BIOS.

> [!info]
> PC manufacturers do not ship computers with BIOS firmware anymore, but the terms "BIOS" and "UEFI" are often used interchangeably, which is a common point of confusion.

[Coreboot](https://www.coreboot.org/end_users.html) is an open-source alternative (not yet used by major brands).
 
### Updates via flashing

"Flashing" is a term used to describe the action of overriding firmware code right in the device's non-volatile memory.

The problem with flashing is that if done uncorrectly it can "brick" your device — the [fwupd](https://github.com/fwupd/fwupd) project aims to make updating firmware on Linux automatic, safe, and reliable.

### Updates via firmware packages

Firmware packages provide binary large objects (blobs) that the Linux kernel loads at runtime — avoiding the need of flashing.

Some common firmware packages are:

- `intel-ucode` / `amd-ucode`
  CPU firmware updates loaded at runtime.
<br>	 
- `linux-firmware`
  Meta-package that pulls in all the vendor-specific sub-packages for GPUs,  wired network adapters,  wireless network adapters,  Bluetooth controllers,  sound cards, etc.

---
 
### Useful commands
	
- `journalctl -kb | grep firmware`
  Print kernel logs from the current boot (with `firmware`).
<br>		
- `dmesg | grep firmware`
  Print kernel messages (with `firmware`).
<br>		
- `lspci -k`
  List PCI devices and their drivers.