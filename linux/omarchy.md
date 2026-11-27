# Omarchy

### Getting started

- No need for dual-boot.
- Omarchy installs so fast that it cannot possibly override the full disk with zeros, find out what they do (maybe just delete the partition table).
- Full-disk encryption:
	- Reuse the encryption password for the user account (single user account)
	- Cannot use Bluetooth keyboard on boot to decrypt disk
- Check the fields that Omarchy ask during installation — maybe put those in top level variables for hosts.
- Languages — no need for other languages, English is enough.


### Coming from Mac of Windows

- This is the part that Omarchy has that is nice and kind of hard to replicate:
	- Omarchy menu — everything is in there, not just a launcher.
	- Screenshots, screen recording — super simple to launch, everything configured ready to be used.
	- Automatic snapshots on every update — this includes normal files, not just system configuration.
- All the advancements that Omarchy makes to support Apple silicon go to **Aurora Silicon** which offers installers for many Linux distros, including NixOS.


### Navigation

It seems that Hyprland is trying to do too much, there are too many layout modes and not all dispatchers work properly in all of them. The Scrolling layout is not even nearly as polished as in Niri.

The compositor part could be easily replaced in other distros with Niri and have a much better personalized experience. Omarchy overly complicate things in order to please broader audiences.


### The top bar

The Omarchy shell is really cool — it is a single process that controls everything: top bar, plugins, menu, notifications, OSD, and the lock screen.

This is the thing that would be really hard to replicate in other distros. It looks simple at first view but it has a lot of attention to detail and is tightly integrated with the rest of the system.

> [!important]
> Customizations to the bar should be done via scripts to allow updates introduce default changes — once you have your own `shell.json`, it’s canonical, there’s no deep merge, so new default widgets in future Omarchy releases won’t appear on your bar automatically. Applying changes via a script allow you to execute `omarchy bar defaults` to apply newer defaults and they apply your changes on top.


### Themes

You have plenty of options here, and appliying a new theme is death simple and fast.

One of the key factors here is the Omarchy shell — being a single tools, everything follows the same style.


### Hotkeys

In Omarchy every hotkey sits on a plane, and each plane has one job:

- `Super + key` for system actions: windows, workspaces, layouts, clipboard, core menus, etc.
- `Super + Shift + key` for launching applications
- `Super + Ctrl + key` for panels and system menus
- `Super + Ctrl + Alt + key` for quick actions and notices


### Unified Clipboard & History

Simple but very useful.


### Reminders

Simple, could be useful.


### Notices

Here you can see the attention to detail. For example, if you click a notification with a verification code, it copied the code to the clipboard but does not save it in history.

Everything can be controlled with the keyboard. Put the focus on the notifications with `Super + Ctrl + Alt + Comma`.


### Text Extraction & Dictation

Text extraction is awesome — the tool behind it is an opensource project in GitHub with a bunch of starts that I have never heard of. This is the leverage of using something that is community driven. The script could be used in other distros as well.

For dictation I prefer Hex — it is free and completely local. use the `Parakeet v3` model for really good english and spanish support.


### Screenshots & Recording

Nice utilities to take screenshots, record the screen (webcam overlay), pick pixel color, capture text, decode QR codes, etc.

Use Tailscale to share between your own devices (locally or remotely), Localsend to share between devices on the same local network and Whatsapp/Signal to share with anyone remotely — you even get the options to transcode videos and photos to lower the resolution before sharing.


### Toggles, Idle and Screensaver

A lot of what you change day to day isn’t really a setting. It’s a mode you flip on for an hour and off again — nightlight, do not disturb, idle, screensaver, etc.


### Omarchy CLI

AI agents can use Omarchy's CLI commands to customize your configuration.

Scripts are divided into groups, check the page for more information.

Interesting, run `omarchy pkg defaults` to show the list of Omarchy default packages.


### Terminal

Nothing fancy. Foot is the default terminal but many other can be used.

Tmux is the default terminal multiplexer (guess this might change in the future).


### Neovim

Intro to neovim. Nothing special.


### AI

One bar icon and one panel for every AI coding subscription on the machine.

In `/usr/share/omarchy/shell/plugins/agents/README.md` you can find a detail explanation of how it works.


### Development tools

Most development tools are installed with `mise` on first use.

I need to learn docker at some point.


### Shell tools

A very short explanation of the included shell tools — found out I really don't need `atuin`, `Ctrl-r` (with fzf) is good enough.


### Shell functions

A short explanation of the shell functions included by default — there are some interesting ones, like `rsw` to use rsync to sync files to another computer, `fip` for ssh port forwarding and more.

I need to learn about Git worktrees.


### TUIs

A brief description of the builtin TUIs.

Any terminal program can get the full app treatment, just run `Install > TUI` from the Omarchy menu.


### GUIs

Builtin GUIs overview. Most Omarchy own apps don't work properly (I guess it is my machine).


### Browsers

Omarchy automatically adjust themes and some plugins from Chromium-based browsers, not for Firefox/Zen.

You can change the default browser but web-apps will always use Chromium, so that is the only browser you cannot uninstall.


### Commercial apps / services



















