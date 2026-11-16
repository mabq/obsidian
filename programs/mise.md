# Mise

- It’s a runtime and tool-version manager — it installs and switches between versions of languages and CLI tools like Node, Python, Go, Ruby, Terraform, or kubectl, and it can pin versions per project through a `mise.toml` file.
- It helps is in the part of your setup that changes most often: the developer toolchains. Use the system package manager for general-purpose tools you use every day and `mise` for language runtimes and project-specific tools that need to match the team’s versions.
- `mise` and the system package manager can overlap — if you have nodejs installed system-wide and also in mise, the one earlier in your `PATH` wins, so remove the pacman copies of anything mise manages.
- Does not hide system tools — works on my machine issue.

## Notes

Most of these tools are answers to one underlying problem: a computer’s state accumulates through a long series of manual actions, and that history is never written down. You install a package, edit a config, add a language runtime, enable a service, and after a year nobody, including you, can say exactly what the machine is or how to rebuild it. Every tool you’ve looked at is an attempt to make that state explicit, reproducible, and changeable without fear.

Several separate problems are bundled into that one.

**Reproducibility.** You want to recreate a machine, or a fleet of them, from a description rather than from memory. NixOS takes this furthest, describing the whole system in one file. chezmoi and mise’s dotfiles feature cover the config layer, and package lists cover the software layer.

**Drift.** Even if you started from a clean description, the live machine diverges as you make changes by hand. A useful tool needs a way to compare the description with reality and either report or correct the difference. Ansible and mise bootstrap status both do this, with different levels of ambition.

**Layers with different owners and change rates.** A single machine contains several layers that change at different speeds and come from different sources: the OS and system packages (slow, from distro maintainers), language runtimes and developer tools (fast, per project), your personal config files (constant, edited by you), and services or secrets (mixed). Each layer historically had its own tool. The reason there are so many tools is that the problem genuinely spans several layers, and no single mechanism fits all of them well. Pacman handles the OS well, nvm and mise handle runtimes well, and chezmoi handles files well. Each tool tends to be strong in one layer and then extends into neighbors, which is why their scopes overlap.

**Versioning and rollback.** Packages and tools change constantly, and an update can break something. Being able to pin exact versions, or go back to a previous state, is a separate concern from simply installing things. This is where lockfiles, snapshots, and atomic system generations come in.

**Imperative versus declarative.** This is the deepest design split. Imperative tools run commands that change the machine (“install this, then edit that”). Declarative tools describe the wanted end state and figure out the steps. Most of the apparent differences between tools come from where each one sits on this spectrum, and from whether it tracks state or just applies it once.

When you’re comparing tools, you can ask a few questions that cut through the method details. Which layer does it manage? Is its source of truth a declared description, a snapshot, or a series of commands? Does it detect drift, and does it correct it? Can you roll back? Who maintains the packages it installs? Answering those usually makes it clear why a tool exists and whether it fits your problem.