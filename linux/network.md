# Network

Think of the network stack as a set of nested layers. Traffic enters physical hardware, gets processed by kernel drivers, passes through routing and filtering logic, and finally reaches your userland applications.

```txt
+--------------------------------------------------------------------+
|  USER SPACE / APPLICATIONS (Browsers, SSH, Nginx, Curl)            |
+--------------------------------------------------------------------+
|  CONTROL PLANE TOOLS (iproute2, Systemd-networkd, NetworkManager)  |
+--------------------------------------------------------------------+
|  KERNEL SPACE: Socket Layer (BSD Sockets)                          |
|  KERNEL SPACE: Transport Layer (TCP, UDP)                          |
|  KERNEL SPACE: Network & Routing (IPv4, IPv6, Subnets)             |
|  KERNEL SPACE: Netfilter / Packet Filtering (Firewalls)            |
|  KERNEL SPACE: Data Link / Network Interfaces (eth0, wlan0)        |
+--------------------------------------------------------------------+
|  HARDWARE / DRIVER (NIC, Wi-Fi Card, Ethernet Cable)               |
+--------------------------------------------------------------------+
```

### User-space tools
 
| Layer / Area | What It Handles | Modern Tools | Legacy Tools |
| --- | --- | --- | --- |
| Physical & Data Link (Layer 1 & 2) | Physical hardware, MAC addresses, link state (up/down), link speed, Wi-Fi. | `ip link`, `ethtool`, `iw` | `ifconfig`, `iwconfig`, `mii-tool` |
| Network & IP Routing (Layer 3) | Assigning IPv4/IPv6 addresses, subnetting, default gateways, routing tables. | `ip addr`, `ip route`, `ip neighbor`, `ping`, `traceroute`, `nc` | `ifconfig`, `route`, `arp` |
| Transport & Sockets (Layer 4) | TCP/UDP ports, open listening sockets, active connections. | `ss -tulnp`, `lsof` | `netstat` |
| Packet Filtering & Firewalls | Dropping packets, NAT (Port forwarding), packet modification. | `nft` (nftables), `ufw`, `firewalld` | `iptables`, `arptables` |
| DNS & Name Resolution | Mapping domain names (`google.com`) to IP addresses. | `dig`, `resolvectl` (systemd-resolved) | `/etc/resolv.conf`, `nslookup` |
| High-Level Network Managers | Automatically switching Wi-Fi, managing VPNs, persisting static configs. | `nmtui`, `nmcli` (NetworkManager), `networkctl` (systemd-networkd) | `/etc/network/interfaces` |

Changes made with the `ip` command are not persistent. For persistent configuration use a network manager.

> [!tip]
> To learn more about these commands check their man pages or help subcommands, e.g. `ip help`, `ip route help` or `man ip`.

A [network manager](https://wiki.archlinux.org/title/Network_configuration#Network_managers) is simply user-space helper tool that automates the process of requesting IP addresses, handling Wi-Fi handshakes, and updating system configuration files for you. The Linux kernel handles network interfaces natively.

Use a Network Manager (e.g. [NetworkManager](https://wiki.archlinux.org/title/NetworkManager)) if you are on a laptop or desktop and frequently move between different Wi-Fi networks, connect to VPNs, or want a system tray applet to click and pick networks.

Skip a Network Manager if you are configuring a headless server, a container, or a router with a fixed static IP or a single Ethernet connection that rarely changes. A lightweight configuration (like [systemd-networkd](https://wiki.archlinux.org/title/Systemd-networkd) or plain static IP config) is much cleaner and faster here.

> [!info]
> Each network interface should be managed by only one DHCP client or network manager, so it is advised to run only one DHCP client or network manager on the system.

### Network Interfaces

To the Linux kernel, a [network interface](https://wiki.archlinux.org/title/Network_configuration#Network_interfaces) is simply a communication endpoint.

Each interface has:

- **State**: `UP`/`DOWN`, and `LOWER_UP` (physical link detected)
- **MTU**: max transmission unit (default 1500 for Ethernet)
- **MAC address**: link-layer hardware address
- **Flags**: `BROADCAST`, `MULTICAST`, `LOOPBACK`, `POINTOPOINT`, etc.
- **Queueing discipline (qdisc)**: how packets are scheduled for transmission

Modern distros use [predictable network interface names](https://systemd.io/PREDICTABLE_INTERFACE_NAMES/) (via `systemd`/`udev`).

#### Physical interfaces

Represent physical NICs (Network Interface Cards). They deal with raw data moving over physical mediums (Ethernet cables or Wi-Fi radio waves). Their job is to get a packet to the router or local LAN.

- `enp3s0` — Ethernet, PCI bus 3, slot 0.
- `wlp2s0` — Wireless, PCI bus 2, slot 0.
- `eno1` — Ethernet, onboard, index 1.
- `lo` — the loopback interface (`127.0.0.1`), always present, used for local communication.

#### Virtual/software interfaces

Exist purely in kernel memory as software:

- `br0` — bridges, which connect multiple interfaces at layer 2 (used heavily in VMs/containers).
- `veth` pairs — virtual Ethernet cables, typically one end in a container's network namespace, the other in the host's (the backbone of Docker/Kubernetes networking).
- `tun`/`tap` — used by VPNs (Tailscale, OpenVPN, WireGuard) to inject packets at layer 3 (tun) or layer 2 (tap).
- `vlan` interfaces (e.g. `eth0.10`) — 802.1Q VLAN tagging.
- `bond0` — link aggregation/bonding, combining multiple NICs for redundancy or throughput.
- `dummy0` — a fake interface, useful for testing or as a stable IP anchor.

Many technologies use virtual interfaces to provide special behaviour. The most common ones are:

| Interface | Software | Purpose |
| --- | --- | --- |
| `tailscale0` | Tailscale | Packet Interception & Encapsulation, Clean IP Isolation & Routing, Security & Firewall Separation. |
| `docker0` | Docker | Bridge network interfaces allow containers on the same host to talk to each other without polluting the physical network. |
| `tun0` / `wg0` | OpenVPN / WireGuard | Encapsulates layer 3 IP packets for standard VPN connections. |
| `virbr0` | KVM / QEMU / libvirt | Acts as a virtual ethernet switch for Virtual Machines running on your computer. |
| `lo` | Linux Kernel | The loopback interface (`127.0.0.1`), allowing your computer to talk to networking services hosted on itself. |

This is how the `tailscale0` interface provides special behavior:

```txt
+-------------------------------------------------------------------+
|                     Your Application (e.g., SSH, Browser)        |
+-------------------------------------------------------------------+
                                  |
            Destination IP: 100.x.y.z (Tailscale IP)
                                  v
+-------------------------------------------------------------------+
| LINUX KERNEL ROUTING TABLE                                        |
| -> "Send 100.x.y.z traffic to tailscale0"                         |
| The kernel thinks tailscale0 is just another network card.        |
+-------------------------------------------------------------------+
                                  |
                                  v
+-------------------------------------------------------------------+
| VIRTUAL INTERFACE: tailscale0 (TUN Device)                       |
| 1. Takes raw IP packet                                            |
| 2. Passes packet to Tailscale daemon                              |
+-------------------------------------------------------------------+
                                  |
                                  v
+-------------------------------------------------------------------+
| TAILSCALE USERSPACE DAEMON                                        |
| 1. Encrypts packet with WireGuard (ChaCha20-Poly1305)             |
| 2. Wraps encrypted payload into a standard UDP packet             |
+-------------------------------------------------------------------+
                                  |
            Destination IP: 192.168.1.1 (Home Router / Internet)
                                  v
+-------------------------------------------------------------------+
| HARDWARE INTERFACE: enp3s0 / wlp2s0                               |
| Sends UDP packet over physical cable/Wi-Fi to remote node         |
+-------------------------------------------------------------------+
```


### Namespaces

Linux network namespaces (`netns`) give each namespace its own isolated set of interfaces, routing tables, and iptables rules. This is the core primitive containers use for network isolation — a container gets its own `veth` interface living in its own namespace, connected back to the host via a bridge.

```bash
ip netns add mynet
ip netns exec mynet ip addr show
```


### Firewalls

`ufw` are `firewalld` are high-level frontends that sit on top of `nftables`/`iptables` to make opening/closing ports easy without writing complex rules manually.


### Concetps

##### CIDR and subnet maks

Finer control of the sizes of subnets allocated to organizations, slowing the exhaustion of IPv4 addresses from the allocation of larger subnets than needed —  the number of addresses of a network may be calculated as `2^(address length − prefix length)`, where address length is `128` for IPv6 and `32` for IPv4. For example, in IPv4, the prefix length `/29` gives: `2^(32−29)` meaning that the subnet can contain `8` IP addresses.