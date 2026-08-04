# Tailscale

Connect devices and services securely across different networks.

Encrypted point-to-point connections (via [WireGuard](https://www.wireguard.com/)) create a peer-to-peer mesh network known as a Tailnet. Can also be used as a traditional VPN by routing all traffic through an [exit node](https://tailscale.com/docs/features/exit-nodes).

Connections between tailnet devices work seamlessly across firewalls and [Network Address Translation (NAT)](https://tailscale.com/blog/how-nat-traversal-works). Not static IP addresses either.

> [!warning]
> Make sure the Tailscale identity provider (e.g. GitHub) has [[bitwarden#^990072|2FA]] enabled.

Packages not intended for other Tailnet nodes are not affected.

### Users and Devices

There are no Tailscale accounts — Tailscale works on top of the [SSO Identity Providers](https://tailscale.com/docs/integrations/identity).

Different Identity Providers produce different Tailnets — e.g. Alice may own a Tailnet linked to GitHub and another one linked to Apple. These are complete different Tailnets.

Users can be invited to partiticipate in other Tailnets — e.g. John can be invited to participate in Alice's Tailnet linked to GitHub.

Each device is linked to a single Tailnet at a time — e.g. John can switch between his own Tailnet and Alice's GitHub Tailnet. The device can not participate on both Tailnets at the same time.

### The Dual-Key Security Model

You are never at the absolute mercy of a foreign Tailnet admin — the ACLs define the maximum boundaries of what could happen, the local client flags and host firewall define what actually happens. For example:

```
 Tailnet Admin (ACLs)             Local Device (Flags / OS)
┌────────────────────────────┐   ┌───────────────────────────┐
│ "Allow SSH to workstation" │ + │ tailscale set --ssh=true  │ = SSH Access Granted
└────────────────────────────┘   └───────────────────────────┘

┌────────────────────────────┐   ┌───────────────────────────┐
│ "Allow SSH to workstation" │ + │ tailscale set --ssh=false │ = Blocked by Local Device
└────────────────────────────┘   └───────────────────────────┘

┌────────────────────────────┐   ┌───────────────────────────┐
│ "Block SSH to workstation" │ + │ tailscale set --ssh=true  │ = BLOCKED by ACL
└────────────────────────────┘   └───────────────────────────┘
```

Same applies for things like `--accept-routes` or `--accept-dns`.


### Tailscale SSH

[Tailscale SSH](https://tailscale.com/docs/reference/syntax/policy-file#tailscale-ssh) completely by-passes the normal SSH authentication system.

> [!warning]
> When connecting to a Tailnet you don't own, use `tailscale up --shields-up` to block all incoming connection attempts from the Tailnet while still allowing you to make outgoing connections to their nodes.

When Tailscale is installed on Linux, macOS, or Windows, its background daemon (`tailscaled`) is registered as a system service executing with `root` / `SYSTEM` privileges — this means it can execute a process and arbitrarily set its user ID (UID) and group ID (GID) to match any existing user account on the system without needing that user's password. <br>
   
```
[Incoming WireGuard Traffic]
       │
       ▼
┌─────────────────────────────────────────┐
│ tailscaled (running as root)            │
│ 1. Verifies signed identity payload     │
│ 2. Validates against tailnet ACL policy │
└──────────────────┬──────────────────────┘
                   │
                   │ Calls PAM / setuid() as root
                   ▼
┌─────────────────────────────────────────┐
│ System Shell Process (bash/zsh)         │
│ UID swapped to target user (e.g., alice)│
└─────────────────────────────────────────┘
```

Because `tailscaled` trusts the signed WireGuard identity payload and the control plane's ACL policy, it doesn't need local OS passwords.


### Documentation

- [Tailscale Docs](https://tailscale.com/docs)
- [How Tailscale Works](https://tailscale.com/blog/how-tailscale-works) — must read article.
- [WireGuard](https://www.wireguard.com/) — the magic of encrypted tunnels.
- [Subnet router](https://tailscale.com/docs/features/subnet-routers/how-to/setup) — access devices not running the Tailscale client.
- [Policies](https://tailscale.com/docs/reference/syntax/policy-file) — avoid unintended access.
- [Tags](https://tailscale.com/docs/features/tags) — tagging a device turns it into a non-user resource, such as a server, NAS, or app connector.
- [Zero trust](https://tailscale.com/docs/concepts/zero-trust)
- [Terminology and concepts](https://tailscale.com/docs/reference/glossary)
- [Features](https://tailscale.com/docs/features)
- [Install Tailscale on NixOS](https://tailscale.com/docs/install/nixos)

