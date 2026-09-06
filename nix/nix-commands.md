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

https://nixos-and-flakes.thiscute.world/best-practices/simplify-nixos-related-commands

https://nixos-and-flakes.thiscute.world/nixos-with-flakes/update-the-system

https://nixos-and-flakes.thiscute.world/nixos-with-flakes/other-useful-tips#viewing-and-deleting-historical-data

