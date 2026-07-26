# Disk setup

```sh
# Wipes filesystem signatures from all (previous) partitions
wipefs -a /dev/sdX[0-9]*
# Wipe the previous partition table
wipefs -a /dev/sdX
```

Create partitions (depends on the receipe)

Encrypt root partition (filesystem exist inside encrypted xxx)

Create filesystems

Mount filesystems




