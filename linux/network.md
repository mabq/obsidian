# Network

Linux abstracts physical network hardware into [network interfaces](https://wiki.archlinux.org/title/Network_configuration#Network_interfaces).

The Linux kernel is in charge. Tools like network managers or the `ip` command simply issue calls to the kernel to set up state.

Read the [Network Configuration](https://wiki.archlinux.org/title/Network_configuration) ArchWiki to learn about Linux networking.


### Network Managers

A [network manager](https://wiki.archlinux.org/title/Network_configuration#Network_managers) lets you manage network connection settings in so called network profiles to facilitate switching networks.

> [!info]
> Each network interface should be managed by only one DHCP client or network manager, so it is advised to run only one DHCP client or network manager on the system.

**1. systemd-networkd**

Primarily declarative, reads network configurtion from static files that are not meant to be edited often.

Use `networkctl help` or see `man networkctl` for network configuration commands.

Use `resolvectl help` or see `man resolvectl` for DNS commands.

**2. network-manager**

Primarily imperative. Provides the commands `nmtui` for easy manual configuration via an interactive TUI and `nmcli` for pure command-line automation, scripting and advanced use.


### Imperative changes

The `ip` command can be used to manage network interfaces, IP addresses and the routing table. These changes are imperative and will be lost after a reboot, for persistent configuration use a network manager.

To learn more about eh `ip` command, see [iproute2](https://wiki.archlinux.org/title/Network_configuration#iproute2) or read its man page `man ip`.


### Concetps

##### CIDR and subnet maks

Finer control of the sizes of subnets allocated to organizations, slowing the exhaustion of IPv4 addresses from the allocation of larger subnets than needed —  the number of addresses of a network may be calculated as `2^(address length − prefix length)`, where address length is `128` for IPv6 and `32` for IPv4. For example, in IPv4, the prefix length `/29` gives: `2^(32−29)` meaning that the subnet can contain `8` IP addresses.