# Tailscale

Connect devices securely across different networks.

[WireGuard](https://www.wireguard.com/) encrypted point-to-point connections create a peer-to-peer mesh network known as a Tailnet — Tailscale automatically handles key generation and distribution, so users benefit by not needing to perform the manual steps.

Tailscale can also be used as a traditional VPN by routing all traffic through an [exit node](https://tailscale.com/docs/features/exit-nodes).

Connections between tailnet devices work seamlessly without static IP addresses across firewalls and [Network Address Translation (NAT)](https://tailscale.com/blog/how-nat-traversal-works).

Packages not intended for other Tailnet nodes are not affected.

### Identity

Each Tailscale connection is based on identity.

Tailscale delegates user authentication to [Identity Providers](https://tailscale.com/docs/integrations/identity#supported-native-identity-providers), it does however establish node identity. A node identity binds a user's identity to a specific device.

> [!warning]
> Prevent malicious takeover of your Tailnet by enabling [[bitwarden#^990072|2FA]] with your Identity Provider.

Tailscale takes into account both the user identity and the node identity when determining what network access to grant.

Different Identity Providers produce different Tailnets. E.g. Alice may own a Tailnet linked to GitHub and another one linked to Apple. These are totally unrelated Tailnets.

Users can be invited to partiticipate in other Tailnets. E.g. John can be invited to participate in one of Alice's Tailnets.

> [!info]
> A user can login into many different Tailnets and switch between them, but a node can connect to only a single tailnet at any given time. For more info read [How devices, nodes and user accounts relate](https://tailscale.com/docs/concepts/tailscale-identity#how-devices-nodes-and-user-accounts-relate) and [Switching between Tailnets](https://tailscale.com/docs/concepts/tailscale-identity#switching-between-tailnets).

For more information read [Tailscale Identity](https://tailscale.com/docs/concepts/tailscale-identity#introduction).

### Tailnet Policy vs. Local Flags

The Tailnet [Policy file](https://tailscale.com/docs/reference/syntax/policy-file) defines the maximum boundaries of what could happen. [Local client flags](https://tailscale.com/docs/reference/tailscale-cli#up) define what actually happens. For example:

```
 Tailnet Policy                   Local Device (Flags / OS)
 --------------------------       -------------------------
 "Allow SSH to workstation"   +   tailscale set --ssh=true    = SSH Access Granted
 "Allow SSH to workstation"   +   tailscale set --ssh=false   = Blocked by Local Flag
 "Block SSH to workstation"   +   tailscale set --ssh=true    = Blocked by Tailnet Policy
```

For the Policy file prefer [Grants](https://tailscale.com/docs/reference/syntax/policy-file#grants) over Legacy [ACLs](https://tailscale.com/docs/reference/syntax/policy-file#acls), even if application layer permissions are not required. 

Use `tailscale debug prefs` to check local client current settings.

> [!warning]
> When connecting to a Tailnet you don't own, use `tailscale up --shields-up` to block all incoming connection attempts from the Tailnet while still allowing you to make outgoing connections to their nodes.


### Tailscale SSH

Tailscale SSH does not require SSH keys or user passwords — the background daemon `tailscaled` is registered as a system service executing with `root` privileges, it can execute a process and arbitrarily set its user ID (UID) and group ID (GID) to match any existing user account on the system **WITHOUT NEEDING THAT USER'S PASSWORD**.

To see why you would prefer Tailscale SSH over normal SSH, read [Tailscale SSH](https://tailscale.com/docs/features/tailscale-ssh) and [Protect your SSH servers using Tailscale](https://tailscale.com/docs/reference/ssh-over-tailscale) — just be careful with the [permissions](https://tailscale.com/docs/reference/syntax/policy-file#tailscale-ssh) set in the [Policy file](https://tailscale.com/docs/features/tailnet-policy-file). 
 
Check if Tailscale SSH is enabled with `tailscale debug prefs | grep RunSSH`.

Enable/disable Tailscale SSH with `sudo tailscale set --ssh=true|false`.

> [!warning]
> Do not enable Tailscale SSH in Tailnets you don't own!


### Tailscale Docs

- [Tailscale Docs](https://tailscale.com/docs)
- [Start using Tailscale](Start using Tailscale) — quickstart, terminology, concepts, etc.
- [How Tailscale Works](https://tailscale.com/blog/how-tailscale-works) — must read article.
- [WireGuard](https://www.wireguard.com/) — the magic of encrypted tunnels.
- [Subnet router](https://tailscale.com/docs/features/subnet-routers/how-to/setup) — access devices not running the Tailscale client.
- [Policies](https://tailscale.com/docs/reference/syntax/policy-file) — avoid unintended access.
- [Tags](https://tailscale.com/docs/features/tags) — tagging a device turns it into a non-user resource, such as a server, NAS, or app connector.
- [Zero trust](https://tailscale.com/docs/concepts/zero-trust)
- [Features](https://tailscale.com/docs/features)
- [Install Tailscale on NixOS](https://tailscale.com/docs/install/nixos)

