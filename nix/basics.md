# Basics


## nixpkgs, NixOS and Home Manager

1. nixpkgs provides both packages AND the module infrastructure.
2. NixOS modules consume packages and define system-wide configuration.
3. Home Manager modules consume packages and define user-specific configuration.
4. Modules share the same evaluation system but have different available options.
5. They can reference each other (e.g., Home Manager can read NixOS config via `osConfig`).
6. nixpkgs is the foundation; everything else builds on it.

```text
┌─────────────────────────────────────────────────────┐
│                      nixpkgs                        │
│  ┌─────────────────────────────────────────────┐    │
│  │  Package Definitions (pkgs.*)               │    │
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
# configuration.nix (NixOS)
{ config, pkgs, ... }: {
  # 1. Install packages globally (nixpkgs layer)
  environment.systemPackages = [ 
    pkgs.bat
    pkgs.git
  ];
  
  # 2. Configure a system service (NixOS module layer)
  services.openssh.enable = true;
  
  # 3. Define a user's Home Manager config
  home-manager.users.alice = { config, pkgs, ... }: {
    # This is the Home Manager layer
    programs.bat.enable = true;
    programs.git.userName = config.nixosConfig.users.users.alice.name;
    # Uses a value from NixOS config
  };
}
```

**Pros and Cons of using modules for configs**

✅ Merges configs from multiple sources
✅ Type checking and validation
✅ Better error messages
✅ Often includes activation scripts (like `bat cache --build`)
✅ Self-documenting
❌ May not support all features
❌ Can be complex for simple configs
❌ Sometimes slower to evaluate
❌ Learning curve for module options


**Pros and Cons of using Using Raw Files**

✅ Full control over content
✅ Use existing dotfiles directly
✅ Simpler for complex configs
✅ Faster to set up
✅ More flexible
❌ No validation
❌ No automatic activation scripts
❌ Manual merging if needed
❌ Less discoverable options

> [!note]
> Neither is "wrong"—it's about what works best for your use case. Start with raw files if you're comfortable, gradually adopt modules when they make your life easier, or stick with raw files forever. All are valid Nix approaches!


---



Using nixos / home-manger options:
- Options are transformed into configuration files.
- The intended approach when using Nix.
- Good: Options that build a single file can be dispersed across many nix modules. E.g.:
  - `environment.sessionVariables` can be used in different modules, each setting the ones it requires.
  - Shell plugins can be initialized from the actual plugin, instead of needing to set that option in the zsh config files.
- Bad: Requires a complete system rebuild even for a small change.
- The nix file contains the actual configurations. What if we want to pick one config from many.

[NixOS - Writting Modules](https://nixos.org/manual/nixos/stable/#sec-writing-modules).
[NixOS - Modules Categorization](https://github.com/NixOS/nixpkgs/blob/master/nixos/modules/module-list.nix)


> [!note]
> You should make package options for your modules, where applicable. While one can always overwrite a specific package throughout nixpkgs by using nixpkgs overlays, they slow down nixpkgs evaluation significantly and are harder to debug when issues arise.






---

Using in-store symlinks:
- Makes a copy of the file the nix store, so it is not modifiable.
- Also requires a complete rebuild for any change.

---

Using out of store symlinks:
- Symlinks to the repository files.
- Changes 