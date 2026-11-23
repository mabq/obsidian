# mynix

### Omarchy to mynix

**Getting started:**

- No need for dual-boot.
- Omarchy installs so fast that it cannot possibly override the full disk with zeros, find out what they do (maybe just delete the partition table).
- Full-disk encryption:
	- Reuse the encryption password for the user account (single user account)
	- Cannot use Bluetooth keyboard on boot to decrypt disk
- Check the fields that Omarchy ask during installation — maybe put those in top level variables for hosts.
- Languages — no need for other languages, English is enough.


**Coming from Mac of Windows:**

- This is the part that Omarchy has that is nice and kind of hard to replicate:
	- Omarchy menu — everything is in there, not just a launcher.
	- Screenshots, screen recording — super simple to launch, everything configured ready to be used.
	- Automatic snapshots on every update — this includes normal files, not just system configuration.
- All the advancements that Omarchy makes to support Apple silicon go to **Aurora Silicon** which offers installers for many Linux distros, including NixOS.


**Navigation:**

It seems that Hyprland is trying to do too much, there are too many layout modes and not all dispatchers work properly in all of them. The Scrolling layout is not even nearly as polished as in Niri.

The compositor part could be easily replaced in other distros with Niri and have a much better personalized experience. Omarchy overly complicate things in order to please broader audiences.


**The top bar:**

The Omarchy shell is really cool — it is a single process that controls everything: bar, plugins, notifications, launcher, menus, lock-screen, etc.

This is the thing that would be really hard to replicate in other distros. It looks simple at first view but it has a lot of attention to detail and is tightly integrated with the rest of the system.

This section made me realize why Omarchy has so many shell scripts — this is what allow you to modify the system programatically!









