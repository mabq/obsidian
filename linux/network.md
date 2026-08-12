# Network

Linux abstracts physical network hardware into Network Interfaces.

The Linux kernel is in charge. Tools like network managers or the `ip` command simply issue calls to the kernel to set up state.

Learn more:
- [Network Configuration](https://wiki.archlinux.org/title/Network_configuration) ArchWiki is an amazing resource to learn about Linux networking.


### Network Managers

A [network manager](https://wiki.archlinux.org/title/Network_configuration#Network_managers) lets you manage network connection settings in so called network profiles to facilitate switching networks.

systemd-networkd is primarily declarative, meaning that it reads network configurtion from static files that are not meant to be edited often. It provides the `networkctl` command to query or modify the status of network links. Use `networkctl help` or see `man networkctl` for more information.

Network-manager is primarily imperative, it provides the commands `nmtui` for easy manual configuration via an interactive TUI and `nmcli` for pure command-line automation, scripting and advanced use.


### Imperative changes

The `ip` command can be used to manage network interfaces, IP addresses and the routing table. These changes are imperative and will be lost after a reboot, for persistent configuration use a network manager.

To learn more about eh `ip` command, see [iproute2](https://wiki.archlinux.org/title/Network_configuration#iproute2) or read its man page `man ip`.


### Concetps

##### CIDR and subnet maks

Finer control of the sizes of subnets allocated to organizations, slowing the exhaustion of IPv4 addresses from the allocation of larger subnets than needed —  the number of addresses of a network may be calculated as `2^(address length − prefix length)`, where address length is `128` for IPv6 and `32` for IPv4. For example, in IPv4, the prefix length `/29` gives: `2^(32−29)` meaning that the subnet can contain `8` IP addresses.