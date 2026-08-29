# Environment variables

> [!important]
> Systemd service environments (system or user) and shell environments are separate namespaces. Setting a variable in your shell (`export FOO=bar`) has zero effect on either `systemctl show-environment` or `systemctl --user show-environment`, and vice versa.


### System manager (system services)

Environment variables available to system services can come from:

1. `DefaultEnvironment=` in `/etc/systemd/system.conf` — sets defaults for all services managed by PID 1.
2. `Environment=` or `EnvironmentFile=` directives inside individual unit files (`[Service]` section).
3. `systemctl set-environment VAR=value` — sets a variable at runtime in the system manager's environment, inherited by subsequently started units.
4. The kernel command line — the only thing "above" PID 1 in terms of environment inheritance.

Check current system manager environment with: `systemctl show-environment`


### User manager (user services)

Each user gets their own `systemd --user` instance, with its own separate environment. Sources include:

1. `DefaultEnvironment=` in `/etc/systemd/user.conf` (or user config drop-ins).
2. `Environment=` or `EnvironmentFile=` in user unit files (`~/.config/systemd/user/*`).
3. `systemctl --user set-environment VAR=value` — sets a variable at runtime in the user manager's environment, inherited by subsequently started units.
4.  `systemctl --user import-environment VAR1 VAR2 ...` — imports the given variables from the current shell into the systemd manager. No arguments imports all variables.

To unser a variable `user systemctl --user unset-environment VAR`.

Check current user manager environment with: `systemctl --user show-environment`.


### Shell environment variables

Variables set by your shell's config files (`/etc/zshenv`, `~/.zshenv`, etc.).

These are only visible to processes launched from the terminal.  Apps launched via a `.desktop` file, app launcher or systemd services won't see them.


### UWSM

When you run `start-hyprland` Hyprland exports some environment variables for the current shell. Hyprland and its child processes get `WAYLAND_DISPLAY`, `HYPRLAND_INSTANCE_SIGNATURE`, etc. in their process environment — but `systemd --user` already started earlier (at login) and knows nothing about these new variables.

Some of thses variables are required by user services (e.g. `WAYLAND_DISPLAY`, `DISPLAY`, etc.), so they must be be explicitly pushed in with `systemctl --user import-environment` and/or `dbus-update-activation-environment`.

UWSM (Universal Wayland Session Manager) exists specifically to formalize this: instead of a bare `start-hyprland`, it launches the compositor as a proper systemd unit (`wayland-wm@.service`) and handles the import step itself, so user services come up with the right environment already synced — and you also get clean `systemctl --user stop` semantics and journal logging for the compositor, instead of it just being an orphaned process under your TTY login shell.


### NixOS session variables

NixOS has an option called `environment.sessionVariables`. You can use it to set environment variables you want available in any shell and systemd user services after login.

> [!info]
> Variables set at runtime like `WAYLAND_DISPLAY` still need to by pushed to systemd user environment. See UWSM above.

Internally these variables are written to `/etc/set-environment`. NixOS automatically configures `/etc/zshenv` (and config files for all other shells) to source that file. It also reads that file and push all those variables to systemd user environment automatically.
