# Nix

### Nix completely rejects the FHS

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


### Git Mental Model

^f8dcd3

If you think about it, NixOS essentially treats your entire operating system configuration exactly like a Git repository checkpoint.

|Git Concept|NixOS Equivalent|How it Works|
|-|-|-|
|`.git/objects/`|`/nix/store/`|The content-addressed database. Just like a Git object hash changes if a single character in a file changes, a Nix store path hash changes if any dependency or configuration line changes. Both are strictly read-only.|
|Commit Hash|Generation|In Git, a commit is a snapshot of your tree. In NixOS, when you run `nixos-rebuild` switch, you create a new system "generation". It is a complete snapshot of your system state.|
|Working Directory|`/run/current-system/sw/` & FHS Symlinks|The symlinks you see in `/bin`, `/run/current-system`, `/etc/profiles/per-user/<USER>` and parts of `/etc` are just the "checked-out files" of your current generation.|
|`git checkout <hash>`|`nixos-rebuild --rollback`|Rolling back in NixOS is literally just pointing those symlinks to an older, already-existing generation directory in the store. It takes less than a second because no files are actually reinstalled.|
















