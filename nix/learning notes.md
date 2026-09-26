# Basics


## Vimjoyer nix course

Best nix course found so far!

- [What is Nix](https://www.vimjoyer.com/course/what-nix-is/) 
- [Nix Language Basics](https://www.vimjoyer.com/course/nix-language/)
- [Sets and decisions](https://www.vimjoyer.com/course/sets-and-decisions/)
- [Functions](https://www.vimjoyer.com/course/functions/)
- [Named function inputs](https://www.vimjoyer.com/course/function-arguments/)
- [Meed nixpkgs](https://www.vimjoyer.com/course/nixpkgs-basics/)
- [NixOS configuration](https://www.vimjoyer.com/course/nixos-basics/)
- [Try nix without installing](https://www.vimjoyer.com/course/nix-cli/)

Nix ecosystem:

- [Nix(OS) Ecosystem Explained](https://www.youtube.com/watch?v=X_jMqi-0SrM) (vimjoyer)

Flakes:

- [Ultimage Nix flakes guide](https://youtu.be/JCeYq72Sko0?t=162) (vimjoyer)
- [Introduction to flakes](https://nixos-and-flakes.thiscute.world/nixos-with-flakes/introduction-to-flakes) (ryan4yin)
- [NixOS Updating | Flakes & Channels](https://www.youtube.com/watch?v=fLICrNK_COw&t=6s) (vimjoyer)

Nix modules:

- [NixOS Module Anatomy](https://www.youtube.com/watch?v=xdDZT1cEuLU)

Nix REPL:

- [Debug Your Nix Code Fast with Nix REPL](https://www.youtube.com/watch?v=swiWnAwionc) (vimjoyer)


## Nix language

Nix describes **one value**, and everything is an expression.

- [Nix Functions Explained](https://www.youtube.com/watch?v=HiTgbsFlPzs) (vimjoyer)
- [Noogle](https://noogle.dev/) (library functions)


## Helpful commands

```sh
# Rebuild flake in cwd
sudo nixos-rebuild switch --flake .#<config-name>`

# Clean
sudo nix-collect-garbage --delete-older-than <N>d # delete system profiles older than N days
sudo nix-collect-garbage -d # remove unused files in the nix store
sudo nixos-rebuild switch # update the boot loader entries
```


## nixpkgs, nixos and home-manager

1. nixpkgs provides both packages and the module infrastructure.
2. NixOS modules consume packages and define system-wide configurations.
3. Home Manager modules consume packages and define user-specific configurations.
4. Modules share the same evaluation system but have different available options.
5. They can reference each other (e.g., Home Manager can read NixOS config via `osConfig`).
6. nixpkgs is the foundation; everything else builds on it.

```text
┌─────────────────────────────────────────────────────┐
│                      nixpkgs                        │
│  ┌─────────────────────────────────────────────┐    │
│  │  - bat, git, nginx, etc.                    │    │
│  └─────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────┐    │
│  │  Module System (lib.modules)                │    │
│  │  - mkOption, mkIf, evalModules              │    │
│  └─────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────┘
                         │
         ┌───────────────┼───────────────┐
         ▼               ▼               ▼
┌─────────────────┐ ┌───────────────┐ ┌─────────────────┐
│   NixOS         │ │   Home        │ │   Other         │
│   Modules       │ │   Manager     │ │   Frameworks    │
│                 │ │   Modules     │ │   (nix-darwin,  │
│ - systemd       │ │ - programs    │ │    devshell,    │
│ - networking    │ │ - home.file   │ │    etc.)        │
│ - users         │ │ - xdg.config  │ │                 │
└─────────────────┘ └───────────────┘ └─────────────────┘
```

Here's how all three layers interact in a real configuration:

```sh
# file: configuration.nix
{ config, pkgs, ... }: {
  # 1. Install packages globally (nixpkgs)
  environment.systemPackages = [ 
    pkgs.bat
    pkgs.git
  ];
  
  # 2. Configure a system service (NixOS)
  services.openssh.enable = true;
  
  # 3. Define a user's Home Manager config
  home-manager.users.alice = { config, pkgs, ... }: {
    # (Home Manager)
    programs.bat.enable = true;
    programs.git.userName = config.nixosConfig.users.users.alice.name;
  };
}
```

Watch [NixOS Module Anatomy](https://www.youtube.com/watch?v=xdDZT1cEuLU).

> [!note]
nixpkgs overlays slow down nixpkgs evaluation significantly and are harder to debug when issues arise.

## Using modules vs. raw files for configs

Both approaches are valid, use what works best for your use case.

#### NixOS/Home-manager modules

The good:
- Merges configs from multiple sources. E.g. set a zsh option from atuin module.
- Type checking and validation.
- Better error messages.
- Often includes activation scripts (like `bat cache --build`).
- Self-documenting.

The bad:
- Requires a full build even for the tiniest change.
- May not support all features.
- Can be complex for simple configs.
- Sometimes slower to evaluate.
- Learning curve for module options.

#### Raw Files

The good:

- Full control over content.
- Use existing dotfiles directly.
- Simpler for complex configs.
- Faster to set up.
- More flexible.

The bad:

- No validation.
- No automatic activation scripts.
- Manual merging if needed.
- Less discoverable options.


## NixOS rebuild steps

When you run `nixos-rebuild` (typically with subcommands like `switch`, `boot`, or `test`), the OS does not modify configuration files in place like a traditional Linux distribution. Instead, it compiles a complete, immutable system state into the Nix Store and then transitions the system to that new state.

**Step-by-Step Execution Flow**

1. Configuration Evaluation (Parsing)
The Nix engine reads your configuration entry point (`/etc/nixos/configuration.nix` or `flake.nix`). It evaluates the Nix language code, resolves imported NixOS modules, and verifies option types. If there is a syntax error or an invalid option, the process fails here immediately without making any changes to your system.
<br>
2. Derivation Building
Once evaluated, Nix produces a derivation file (`.drv`). This acts as a blueprint describing the exact dependencies, build scripts, packages, and configuration files needed to build the new system version.
<br>
3. Realization in `/nix/store` (Building)
Nix downloads (`nixos.cache.org`) or builds all necessary dependencies locally. A new directory is created at `/nix/store/<hash>-nixos-system-...` containing the full system closure:
   - The Linux kernel and kernel modules.
   - Activation scripts.
   - Configuration files for `/etc`.
   - Symlinks to all installed package binaries.
<br>
4. Generation Registration
The newly built system path is linked into the system profile directory at `/nix/var/nix/profiles/system`. The profile generation number is incremented (e.g., from generation 42 to 43), updating the current system pointer.
<br>
5. Bootloader Update
The bootloader installation script runs (e.g., for GRUB or systemd-boot). It adds or updates boot entries so you can select the new generation (or roll back to any previous generation) at boot time.
<br>
6. Live Activation (for `switch` or `test`)
The activation scripts run to apply changes on the live system:
   - `/etc` symlinks are updated to point to the new paths in `/nix/store`.
   - NixOS compares old systemd units with the new ones. Modified or removed services are stopped or reloaded, and new services are started without requiring a full system reboot.


### NixOS git analogy

^f8dcd3

NixOS fundamentally treats the entire operating system configuration like a Git repository, viewing the entire filesystem as a declarative "checkout" rather than a mutable pile of files.

For example:

- The `/nix/store` is just like the `.git/objects/` database.
  Files on both directories are immutable. Git creates a new object (with a new hash) when anything changes in a file. Nix creates a new entry in the `/nix/store` with a new hash when a single dependency or configuration line is altered.
<br>
- System generations are just like git commits.
  Every time you do a `nixos-rebuild`, you are essentially "committing" a new system generation. It is a complete, immutable snapshot of your system state.
  - `/nix/var/nix/profiles` - all system generations.
  - `/run/current-system` - symlink to the current system generation.
  - `/etc/profiles/per-user/<USER>` - symlink to the current user profile.
   
  Nix Store objects are kept until you decide to delete the old generations. Elements that are not referenced by any generation are garbage-collected.
<br>
- The FHS resembles a git working directory
  Directories like `/etc` or `/run/current-system` are not filled with actual files; they are a working directory tree composed of symlinks pointing directly into the immutable `/nix/store`.
<br>
- `nixos-rebuild --rollback` is just like `git checkout <hash>`
  Rolling back to a previous system state at boot or via the CLI doesn't uninstall or reinstall anything. It instantly flips the root symlinks to point back to an older, intact store generation.
<br>
- `sudo nix-collect-garbage -d` is just like doing a `git gc`
  Unused packages and old configurations are safely left isolated in the store until you explicitly run the garbage collector to prune orphaned objects.


## Nix completely rejects the FHS

^697838

While traditional Linux distributions rely on the [File System Hierarchy Standard](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard) (FHS) to keep the system organized, Nix views the standard as a fundamental blocker to reproducible, reliable package management.

The FHS assumes a global, shared environment where all applications share the same folders for binaries, libraries and configurations, making reproduceability impossible.

When you compile a program on a standard Linux distro, the compiler automatically looks in `/usr/lib` or `/lib` to find dynamic libraries (like `libc`).

This creates a massive problem for reproducibility:

- If you update a library in `/usr/lib`, you might accidentally break three other programs that depended on the old behavior.
- If a package build succeeds on your machine because you happen to have a random library installed in `/usr/local/lib`, it will fail on a clean machine that lacks it.

Nix abandons this global sharing. Instead, it treats the filesystem as a giant, immutable dependency graph.

By using `/nix/store` and encoding a cryptographic hash of all inputs (source code, compiler versions, dependencies) directly into the directory name, Nix ensures that a package only looks at its explicit, **isolated dependencies**.


|Feature|Linux FHS| Nix Store |
|-|-|-
|Binaries Location|`/bin`, `/usr/bin`, `/usr/local/bin`|Unique paths like `/nix/store/md5sh...-bash-5.2/bin`|
|Libraries Location|`/lib`, `/usr/lib`|Isolated inside each package's individual store directory|
|State|Mutable — packages overwrite or share files in these directories|Immutable — once a store path is built, it is read-only.|
|Multi-version Support|Difficult — leads to "dependency hell" if two apps need different versions of e.g. `openssl`|Native — 10 versions of `openssl` can coexisting perfectly fine.|

If you download a generic Linux binary (like an official executable from a website) and try to run it on NixOS, you will usually get a confusing error like: `bash: ./my-program: No such file or directory`.

This happens because the binary has a hardcoded path to the dynamic linker (the program interpreter) expected by the FHS, usually `/lib64/ld-linux-x86-64.so.2`. On a pure NixOS system, that path literally does not exist.

When dealing with proprietary software or tools that strictly expect an FHS environment, Nix uses a few clever workarounds:

- `patchelf`: Nixpkgs developers use a tool called patchelf to modify the headers of pre-compiled binaries. It rewrites the hardcoded FHS paths to point directly to the correct library locations inside `/nix/store`.
- FHS Environments (`buildFHSUserEnv` / `appimage-run`): Nix can spin up an isolated, temporary container-like environment using Linux namespaces. Inside this bubble, a mock FHS layout (`/bin`, `/lib`, `/usr/lib`) is dynamically mapped out of Nix store paths, allowing unmodified binaries (like Steam games or VS Code extensions) to run smoothly.

If you are running the Nix package manager on top of a standard distribution like Ubuntu or Fedora, Nix lives peacefully inside its `/nix` folder and leaves your system's host FHS completely intact.

