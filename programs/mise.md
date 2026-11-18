# Mise

`mise` started strictly as a **Polyglot Runtime Manager**—handling versions of Node.js, Python, Go, Ruby, and Java per project.

However, because developers always need to orchestrate surrounding tools, tasks, and environments, `mise` expanded its scope. It now functions as a unified developer experience (DevEx) platform that bridges three distinct contexts: **Runtime Management**, **Task Running**, and **System Bootstrapping/Dotfiles**.

The differences across these contexts, what problems they solve, and how they relate to traditional tools can be broken down as follows:

---

### 1. Runtime Version Management (The Core)

* **What it does:** Manages toolchains and language versions (e.g., ensuring project A uses Node v20 and project B uses Node v18).
* **How it works:** It reads files like `.tool-versions` or `mise.toml` and automatically switches runtimes when you `cd` into a directory using **shims** or environment paths.
* **Replaces:** `asdf`, `nvm`, `pyenv`, `rbenv`, `sdkman`, etc.

### 2. Task Running (The Glue)

* **What it does:** Defines and runs project-specific workflows, shell scripts, or build steps.
* **How it works:** You define tasks inside your `mise.toml` using simple commands or scripts. Because `mise` already manages your runtimes, it guarantees that any dependencies required by those tasks (like a specific compiler or CLI tool) are automatically available in the execution path.
* **Replaces:** `Makefile`, `Taskfile`, `npm scripts` (when used across mixed-language codebases), or custom shell scripts.

### 3. System Bootstrapping & Dotfiles (The New Frontier)

* **What it does:** Takes a fresh, blank operating system (macOS or Linux) and provisions it into a fully configured development machine using declarative configuration.
* **How it works:** Through its `mise bootstrap` and `[dotfiles]` features, `mise` can:
* Install system packages and Homebrew formulae/casks.
* Create system directories and configuration files with precise file permissions (`[bootstrap.files]` and `[bootstrap.directories]`).
* Manage dotfiles by symlinking, copying, or tracking them across machines (`[dotfiles]`).
* Tweak OS-level preferences (like macOS `defaults`).


* **Replaces:** Traditional dotfile managers like **GNU Stow**, or heavy system-provisioning frameworks like **Ansible**, **Chef**, or parts of **Nix/nix-darwin** (though lighter weight and less sandboxed than Nix).


### Why use `mise` for all of this?

The major advantage is **consolidation**. Historically, a developer's machine required `nvm` for Node, `brew` for CLI packages, `stow` for dotfiles, and a custom `Makefile` to glue everything together. By pulling system bootstrapping and dotfiles into `mise`, you can theoretically take a brand-new computer, install `mise` once, clone your dotfiles repository, and run a single command to rebuild your entire workspace with your tools, configuration files, and preferences intact.

---

## Tips

Use `[bootstrap.packages]` by default and fallback to `[tools]` for when you need the tool to change versions between projects or it the system package manager does not provide the version you need.
