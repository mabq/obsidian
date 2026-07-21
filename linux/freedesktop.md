# freedesktop.org

While Linux at its core is just an operating system kernel, the ecosystem relies on shared standards to ensure that different graphical interfaces, software packages, and system components work seamlessly together.

[freedesktop.org](https://www.freedesktop.org) (originally called the X Desktop Group, which is why many of its specifications use the prefix **XDG**) is the primary organization enabling interoperability across different desktop environments like GNOME, KDE Plasma, and Xfce. 


### Key Standards from freedesktop.org

Rather than forcing every desktop environment to reinvent the wheel, freedesktop.org establishes common [specifications](https://www.freedesktop.org/wiki/Specifications/) so applications look and behave consistently regardless of which desktop you run. 

- XDG Base Directory Specification
  Defines standard paths for application data, configuration files, and temporary caches (e.g., `$XDG_CONFIG_HOME` defaulting to `~/.config`, `$XDG_DATA_HOME` to `~/.local/share`, and `$XDG_CACHE_HOME` to `~/.cache`). 
<br>
- Desktop Entry Specification (`.desktop` files)
  Standardizes how application launchers are defined, allowing your application menu to parse metadata like app names, icons, category tags, and launch commands. 
<br>
- Shared MIME Info & File Associations
  Rules for how the system identifies file types (MIME types) and determines which default application opens a given file. 
<br>
- Icon Theme Specification
  Standardizes directory layouts and naming conventions for icon sets so themes apply system-wide across different toolkits (GTK, Qt, etc.).


### Beyond specifications

freedesktop.org host fundamental projects like **D-Bus** (inter-process communication), **Wayland** (modern display protocol), **Mesa** (open-source graphics drivers), **Fontconfig**, and **PipeWire** (audio/video routing).

