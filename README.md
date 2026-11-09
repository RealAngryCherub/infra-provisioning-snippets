# Infra Provisioning Snippets

Personal home-lab infrastructure: VLAN-segmented networking, Proxmox virtualization, Caddy reverse proxy, and Tailscale mesh VPN. All personal IPs, domains, and credentials stripped — these are annotated config patterns, not a live blueprint.

## What's Here

```
pfSense/
├── vlan-topology.md       # 7-zone 802.1Q VLAN layout and DHCP scopes
├── firewall-rules.md      # Default-deny interface rules and inter-zone policies
└── openvpn-config.md      # Certificate-based remote access setup notes

proxmox/
├── vm-deployment-notes.md # VM creation patterns (qm, cloud-init)
└── network-bridge.md      # VLAN-aware bridge config for guest isolation

caddy/
└── Caddyfile.sample       # Reverse-proxy blocks with loopback binding

tailscale/
├── mesh-topology.md       # Subnet router, ACL notes, tailnet layout
└── serve-patterns.md      # Exposing loopback services via Tailscale Serve
```

## Zone Map

| VLAN | Purpose | Trust Level |
|------|---------|-------------|
| 10   | Management (Proxmox, pfSense admin) | Internal only |
| 20   | Servers (Active Directory, DNS, Wazuh) | Restricted |
| 30   | User devices / Wi-Fi | Moderate |
| 40   | Guest / IoT (isolated) | Low / no LAN access |
| 50   | Agent workers (isolated automation nodes) | No outbound except SOCKS5 |
| 60   | VOIP | Media only |
| 80   | DMZ (public-facing services via reverse proxy) | Exposed to WAN |

Each SSID maps 1:1 to its VLAN. No implicit trust between zones.

## Disclaimer

These configs are **sanitized reference snippets** from a live home-lab environment. They are not a turnkey deployment — adapt to your own addressing scheme, interface names, and security requirements before using anywhere.

## Links

- **Notion Portfolio:** [zdrouilhet.notion.site](https://app.notion.com/p/Zachary-Drouilhet-IT-Infrastructure-Engineering-Portfolio-018e60e7ab544c2180f6348c6df99b9d?source=copy_link)