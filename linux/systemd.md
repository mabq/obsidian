# systemd

[systemd](https://wiki.archlinux.org/title/Systemd) is the init system and service manager used by most major Linux distributions.

It is the first process the kernel starts and it's responsible for bringing the rest of the system up, managing services, and staying in control throughout the machine's runtime. It replaced older init systems like SysVinit and Upstart, offering parallelized startup, dependency-based ordering, and a unified toolset for system management.

Its main features are:

- **Service management**
  Defines and controls services via declarative `.service` unit files (start, stop, restart, enable, disable). Managed through `systemctl`.
<br>
- **Parallelized boot & dependency resolution**
  Starts services concurrently based on declared dependencies rather than a fixed sequential order, drastically speeding up boot time.
<br>
- **Process supervision & automatic restart**
  Tracks service processes via `cgroups` and can automatically restart failed services, preventing zombie/orphaned processes.
<br>
- **Logging**
  Centralized, structured binary logging system (`journalctl`) that captures kernel, boot, and service logs in one place.
<br>
- **Target units**
  Systemd's version of old-school SysV runlevels. They're synchronization points that group units together and represent a state the system reaches during boot (e.g., `multi-user.target`, `graphical.target`).
<br>
- **Socket, path, and timer activation**
  Services can be started on-demand (socket activation), when a file/path changes, or on a schedule (`.timer` units, replacing/complementing cron).
<br>
- **Dependency types**
  Fine-grained control over ordering and dependency strength between units (Wants, Requires, After, Before).
<br>
- **cgroups integration**
  Uses Linux control groups to track and limit resource usage (CPU, memory, I/O) per service.
<br>
- **Device & mount management**
  Manages hardware device events and filesystem mounts declaratively (`udev` integration, `.mount`/`.automount` units).
<br>
- **Network configuration**
  Optional lightweight network management daemon (`systemd-networkd`).
<br>
- **Name resolution**
  Handles DNS resolution and caching (`systemd-resolved`).
<br>
- **User session management**
  Manages user logins, sessions, and seats (`loginctl`).
<br>
- **Timers as cron alternative**
  Already mentioned above, but worth noting as a standalone convenience feature for scheduled tasks.
<br>
- **Containers/namespaces tooling**
  Lightweight container and chroot-like environment management (`systemd-nspawn`, `machinectl`).
<br>
- **Boot analysis tools**
  Diagnose and profile boot performance, unit dependency trees, etc. (`systemd-analyze`).


## systemd services

Virtually any executable program or script can be run as a systemd service by creating a `.service` unit file.

Whether it is a compiled C binary, a Python script, a Bash command, or a web server, systemd simply needs to know the exact path to the executable and how to manage its lifecycle.

To run a program as a systemd service, it must fulfill two basic rules:

1. **Absolute executable path**
   You must provide the absolute path to the binary or interpreter (e.g., `/usr/bin/python3` or `/nix/store/.../bin/python3`), as systemd does not search your user's `$PATH`.
<br>
2. **Execution permissions**
   The file must be executable (`chmod +x /path/to/script`).
<br>

**Standard Unit File Structure**

System services are placed in `/etc/systemd/system/myapp.service`:

```ini
[Unit]
Description=My Custom Application
After=network.target

[Service]
Type=simple
User=nobody
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/python3 /opt/myapp/main.py
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target

```

In systemd, `wantedBy = [ "multi-user.target" ]` is a directive that tells systemd to automatically start your service when the system reaches the `multi-user.target` state.

When you enable a service with this setting (either by running `systemctl enable` or via a NixOS module), systemd creates a symbolic link to your service file inside the `/etc/systemd/system/multi-user.target.wants/` directory . When `multi-user.target` is started during boot, systemd reads this directory and starts all services linked from it.

<br>

**Other Common Values for `wantedBy`**

The value you use should correspond to the target you want your service to be a dependency of. Here are some of the most common alternatives:

| **wantedBy Value** | **Use Case / When to Use** |
| :--- | :--- |
| `multi-user.target` | **Default for system services** that should start at boot in a non-graphical environment . |
| `graphical.target` | For services that require or are strongly associated with a graphical desktop environment (e.g., a display manager) . |
| `default.target` | **Used for user services** (`systemd.user.services`) to make them start when a user logs in . |
| `sockets.target` | **Used for socket-activated services.** The service itself is not started at boot, but its associated socket unit is, which will then start the service on-demand . |
| `sysinit.target` | For services that need to be started **very early in the boot process**, such as system initialization or rescue/single-user mode services . |
| `shutdown.target` | Although not typically used with `WantedBy`, it's worth noting that all targets (like `multi-user.target`) automatically gain a `Conflicts=` and `Before=` dependency on `shutdown.target` to ensure they are stopped during system power-off . |

<br>

**Edge Cases & Important Caveats**

While *any* binary can run, different types of applications require specific unit file configurations:

* **Graphical applications**
  GUI programs (like Firefox or VLC) expect a display server (`$DISPLAY` or `$WAYLAND_DISPLAY`) and an active user session. While possible, running GUI programs via systemd requires running them as a **systemd user service** (`systemctl --user`) within your active graphical session rather than a system-wide service.
  <br>
* **Forking / Daemonizing legacy apps**
  If a legacy program automatically forks itself into the background when launched, set `Type=forking` in the unit file so systemd tracks the correct child process.
  <br>
* **Short-Lived scripts & Cron alternatives**
  For scripts that run a task and immediately exit (rather than running continuously in the background), set `Type=oneshot`. You can pair them with a `.timer` unit to replace `cron` jobs.
  <br>
* **Interactive terminal applications**
  Programs that require interactive keyboard input (`stdin`) generally cannot run cleanly as background services unless attached to a virtual terminal device (`TTYPath=`) or run inside a terminal multiplexer like `tmux`.

<br>

**Basic workflow to enable any service**

1. Create the file: `sudo nvim /etc/systemd/system/<service-name>.service`
2. Reload systemd configuration: `sudo systemctl daemon-reload`
3. Start the service: `sudo systemctl start myapp`
4. Check status & logs: `sudo systemctl status myapp` or `journalctl -u myapp`
5. Enable auto-start on boot: `sudo systemctl enable myapp`

<br>

**Terminal launch vs. systemd service** ^92b7a6

When you launch a program from a terminal, it runs attached to your temporary desktop session. When you launch it as a systemd service, it runs as an independent system process managed directly by PID 1.

| Feature | Terminal Launch | systemd Service |
| --- | --- | --- |
| **Parent Process** | Your shell (`bash`, `zsh`, etc.) | systemd (PID 1) |
| **Lifespan** | Dies when terminal closes or user logs out (unless run with `nohup` or `tmux`) | Persists across logouts and auto-starts on boot |
| **Environment** | Inherits your current shell env (`PATH`, user env vars, working directory) | Runs in a minimal, isolated environment specified in `.service` file |
| **Crash Recovery** | Stops running; requires manual user restart | Automatically restarts based on configuration (`Restart=on-failure`) |
| **Logs & Output** | Dumped directly to standard terminal output (`stdout`/`stderr`) | Intercepted and indexed by `journalctl` |
| **User Privileges** | Runs under your logged-in user account | Can easily be assigned specific users, groups, or isolated sandboxes |
| **Resource Limits** | Shares limits of your interactive shell session | Enforces strict CPU, memory, and task limits via `cgroups` |

**1. Process Hierarchy & Lifecycle**

* **Terminal:** The shell forks a sub-process attached to a pseudo-terminal (`tty`). Closing the terminal window sends a Hangup signal (`SIGHUP`) to the process, killing it unless you detached it.
* **systemd Service:** The process runs as a child of systemd. It stays active completely independently of whether any user is logged in.

**2. Environment & Context**

* **Terminal:** Programs rely heavily on shell initialization files (`.bashrc`, `.profile`). If a program relies on `$PATH` or custom export variables set in your shell, it works seamlessly.
* **systemd Service:** Does not load shell profiles. Paths to binaries must either be absolute (e.g., `/usr/bin/python3`) or explicitly defined using `Environment=` directives in the unit file.

**3. Resource Control & Security**

* **Terminal:** Has access to your user permissions and home directory, but lacks built-in isolation from other programs running under your account.
* **systemd Service:** Leverages Linux `cgroups` (control groups) and namespaces. You can easily restrict a service to prevent access to `/home`, mark `/etc` read-only, or limit its maximum memory consumption.

[^1]: 
