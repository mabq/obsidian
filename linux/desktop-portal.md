# Desktop portal

A Desktop Portal (like `xdg-desktop-portal`) is a standardized background bridge that lets applications safely ask the desktop environment to perform privileged tasks — like taking a screenshot, sharing your screen, opening a file picker, or querying the dark mode setting.

Wayland compositors operate under a strict security model compared to legacy X11.

- **Wayland's Security Isolation**
  Under X11, any application could secretly capture your screen, spy on keystrokes, or read windows belonging to other apps. Wayland intentionally blocks applications from seeing outside their own window. Because an app like OBS Studio cannot capture your screen directly, it must ask the Wayland Compositor via a desktop portal.
<br>
- **Standardized Screen Sharing**
  Portals provide a universal D-Bus API (org.freedesktop.portal.ScreenCast). Apps like Firefox, Chrome, or Discord do not need custom code to talk to Niri, Sway, Hyprland, or GNOME; they simply talk to `xdg-desktop-portal`.
<br>
- **Sandboxed Apps**
  Modern sandboxed applications (Flatpak/Snap) are locked in containers with no access to your files or hardware. When a Flatpak app wants you to pick a file, the portal pops up a native system file picker outside the sandbox and passes only the selected file back inside.
<br>
- **Modularity**
  Niri/Hyprland is just a window manager/compositor — it does not ship with built-in file choosers, screenshot popups, or app choosers. It relies on portal backends (like `xdg-desktop-portal-gnome` or `xdg-desktop-portal-gtk`) to render those interface prompts.

Without a functioning portal implementation running alongside the compositor, features like Discord screen sharing, OBS display capture, dark theme auto-switching, and Flatpak file dialogs will fail or hang indefinitely.


### Multiple desktop portals

Different desktop portal backends cover different features.

`xdg-desktop-portal` acts as a central frontend router. When an application (like Firefox, OBS, or a Flatpak) requests a system service —such as screen sharing, picking a file, or reading light/dark theme preferences— the frontend looks up which backend handles that specific interface.

No single compositor portal implements every single interface.

<br>

**The Division of Labor:**

**1. Compositor-Specific Backend**

* `xdg-desktop-portal-hyprland` or `xdg-desktop-portal-gnome`/`wlr`/`generic`.
* **What it handles:** Actions that require direct, deep integration with the display server.
* **Primary tasks:** Screen sharing (Screencast via PipeWire), window/output captures (Screenshot), and custom global keyboard shortcuts.
* **Why it's unique:** A generic toolkit portal cannot grab Hyprland's or Niri's internal framebuffers or window metadata directly. That has to be served by a backend that speaks the compositor's specific protocols.

**2. Generic / Fallback Backend**

* `xdg-desktop-portal-gtk` or `xdg-desktop-portal-kde`.
* **What it handles:** Standard GUI dialogs and desktop environment settings.
* **Primary tasks:** Native file chooser dialogs (Open/Save File), AppChooser, print dialogs, and System Settings (like system-wide dark mode or font scaling).
* **Why it's unique:** Compositor developers don't want to re-invent complex UI dialogs like file pickers, so they don't build them into `xdg-desktop-portal-hyprland`. Instead, they delegate those interfaces to a toolkit-based portal.

<br>

**How Portals Are Configured:**

`xdg-desktop-portal` reads a configuration file (like `~/.config/xdg-desktop-portal/portals.conf` or distro defaults) to route traffic.

For instance, a typical Hyprland setup delegates screen recording to Hyprland's portal while routing file dialogs to GTK:

```ini
[preferred]
default=gtk
org.freedesktop.impl.portal.ScreenCast=hyprland
org.freedesktop.impl.portal.Screenshot=hyprland

```

If you only install the compositor's specific portal backend, screen sharing will work, but apps trying to trigger file pickers or theme queries will hang or fail because no backend is registered to answer those D-Bus calls.

<br>

**Learn more:**

- [XDG Desktop Portal](https://wiki.hypr.land/Useful-Utilities/Must-have/#xdg-desktop-portal) (Hyprland)
- [Portals](https://niri-wm.github.io/niri/Important-Software.html#portals) (Niri)


