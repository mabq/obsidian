# Nix commands

```sh
# Rebuild flake in cwd
sudo nixos-rebuild switch --flake .#<config-name>`

# Clean
#   Delete system profiles older than N days
sudo nix-collect-garbage --delete-older-than <N>d
#   Remove unused files in the nix store
sudo nix-collect-garbage -d
#   Update the boot loader entries
sudo nixos-rebuild switch
```


