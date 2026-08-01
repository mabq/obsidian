# Tailscale

Connect nodes in a Tailnet directly and securily — no static IPs, NAT configs, firewall configs or WireGuard configs required.

Packages not intended for other Tailnet nodes are not touched by the Tailscale client in any way.

For a very nice instruduction, read [How Tailscale Works](https://tailscale.com/blog/how-tailscale-works).

### Overview

> [!warning]
> Make sure the Tailscale identity provider (e.g. GitHub) has [[bitwarden#^990072|2FA]] enabled.


### Helpful links

- [WireGuard](https://www.wireguard.com/) — the magic of encrypted tunnels
- [What is Tailscale](https://tailscale.com/docs/concepts/what-is-tailscale)
- [Tailscale quickstart](https://tailscale.com/docs/how-to/quickstart)
- [Exit node](https://tailscale.com/docs/features/exit-nodes/how-to/setup) — Tailscale as a traditional VPN, routing all traffic through a single node (e.g. AppleTV).
- [Subnet router](https://tailscale.com/docs/features/subnet-routers/how-to/setup) — access devices not running the Tailscale client.
- [Policies](https://tailscale.com/docs/reference/syntax/policy-file) — avoid unintended access.
- [Tags](https://tailscale.com/docs/features/tags) — tagging a device turns it into a non-user resource, such as a server, NAS, or app connector.
- [Tailscale SSH](https://tailscale.com/docs/reference/syntax/policy-file#tailscale-ssh) — let Tailscale manage the authentication and authorization of SSH connections in your tailnet.

For more information, visit:

- [Terminology and concepts](https://tailscale.com/docs/reference/glossary)
- [Tailscale CLI](https://tailscale.com/docs/reference/tailscale-cli)
- [Features](https://tailscale.com/docs/features)
- [Install Tailscale on NixOS](https://tailscale.com/docs/install/nixos)

