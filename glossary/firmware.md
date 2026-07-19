# Firmware

Low-level software embedded in hardware that controls basic device functions.

> [!tip]
> Buy hardware from "trusted" brands and hope for the best.

Firmware is the very first code to execute on the system, so it runs in privileged mode on the main CPU with full access to all RAM.

Closed-source firmware (almost all UEFI, BIOS, SSD, GPU, etc.) creates serious trust and supply-chain risks — companies can hide backdoors, bugs, or intentional flaws with no audit. Many experts consider it one of the biggest security weaknesses in modern PCs. For more info visit [Firmware Security Risks (Wikipedia)](https://en.wikipedia.org/wiki/Firmware#Security_risks).

## Motherboard firmware

Two options:

- UEFI (Unified Extensible Firmware Interface)
- Legacy BIOS

To identify the motherboard firmware check the directory `/sys/firmware/efi`, if it exists the firmware is UEFI, if it does not the firmware is BIOS.

> [!info]
> PC manufacturers do not ship computers with BIOS firmware anymore, but the terms "BIOS" and "UEFI" are often used interchangeably, which is a common point of confusion.

[Coreboot](https://www.coreboot.org/end_users.html) is an open-source alternative (not yet used by major brands).

## Updates

### Flashing (less common)

Updates by writing new code in the device's non-volatile memory.

> [!warning]
> Can brick your device — ensure stable power before proceeding. 

`fwupd` is a system service which connects to [LVFS](https://fwupd.org/) (Linux Vendor Firmware Service) to check for firmware updates to a wide variety of hardware from multiple vendors.

### Firmware packages

These packages add blobs (binary large objects) to `/lib/firmware` that the kernel loads at runtime. Some common firmware packages are:

`intel-ucode` / `amd-ucode`
	CPU firmware updates loaded at runtime.
	 
`linux-firmware`
	Meta-package that pulls in all the vendor-specific sub-packages for GPUs,  wired network adapters,  wireless network adapters,  Bluetooth controllers,  sound cards, etc.
  
## Commands
	
`journalctl -kb | grep firmware`
	Print kernel logs from the current boot (with `firmware`).
		
`dmesg | grep firmware`
	Print kernel messages (with `firmware`).
	
`lspci -k`
	List PCI devices and their drivers.