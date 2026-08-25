# D-Bus

D-Bus (Desktop Bus) is an internal messaging postal service for Linux and Unix-like operating systems. It allows different software programs running at the same time to talk to each other and share information without needing to know anything about how the other application was built.

It is an independent, user-space specification and software project maintained by the freedesktop.org community. It relies on standard Linux kernel features (specifically Unix domain sockets) to transport messages between processes.

Systemd relies on D-Bus heavily to control services, manage user sessions, and handle system hardware events. systemd includes its own C-library implementation (sd-bus) to communicate over D-Bus, and running systemd automatically starts D-Bus background services for you. Because they ship together on most distributions, people often mistake D-Bus for a systemd sub-component.

Desktop environments and standalone Linux applications use D-Bus to pass notifications, media keys, status indicators, and screen-sharing requests back and forth.


### How It Works

Instead of forcing applications to open direct custom connections between one another, D-Bus operates using two main communication channels (called "buses"):

- **The System Bus**
  Handles low-level system events. It tracks hardware state changes—like plugging in a USB drive, connecting to Wi-Fi, or dropping battery power—and alerts the system.
<br>
- **The Session Bus**
  Handles user desktop tasks. It links desktop applications logged in under your current session—like letting your media player talk to your volume control widget, or letting a browser send a desktop pop-up notification.


### Why Operating Systems Use It

- **Decoupling**
  A web browser doesn't need custom code to talk to GNOME, KDE, or XFCE notification systems; it just fires a standardized "send notification" D-Bus message.
<br>
- **Security & Permissions**
  The central system bus enforces safety rules so standard user programs can't ask the operating system to perform dangerous administrative actions.
<br>
- **Efficiency**
  Applications can broadcast events (signals) once, allowing any interested program to listen without keeping heavy polling background loops open.
