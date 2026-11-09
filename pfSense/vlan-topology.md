# VLAN Topology — 7-Zone Segmentation

## Interface Layout

- **WAN:** dhcp (ISP-assigned)
- **LAN (trunk to switch):** em1 / vtnet0
- **VLAN interfaces** are created as sub-interfaces on the LAN trunk

## VLAN-to-Subnet Mapping

| VLAN | Subnet     | Description                | DHCP Pool           |
|------|------------|----------------------------|---------------------|
| 10   | 10.x.x.0/24 | Management (hosts, admins) | .100–.150           |
| 20   | 20.x.x.0/24 | Servers (AD, DNS, Wazuh)   | .50–.100            |
| 30   | 30.x.x.0/24 | User devices / trusted Wi-Fi | .100–.200           |
| 40   | 40.x.x.0/24 | Guest Wi-Fi / IoT          | .10–.50             |
| 50   | 50.x.x.0/24 | Agent workers (isolated)   | .100–.150           |
| 60   | 60.x.x.0/24 | VOIP                       | .10–.30             |
| 80   | 80.x.x.0/24 | DMZ (reverse-proxied services) | .10–.50          |

## SSID-to-VLAN Mapping (Wireless AP)

SSIDs on the ASUS RT-BE86U map 1:1:

| SSID              | VLAN | Notes                    |
|-------------------|------|--------------------------|
| Home-Management   | 10   | Admin access only        |
| Home-Trusted      | 30   | Main user devices        |
| Home-Guest        | 40   | Guest / IoT / no LAN     |
| Home-Agent        | 50   | Hermes worker nodes only |

## DHCP Configuration

- pfSense DHCP server handles all VLANs except 50 (static assignments only for worker nodes)
- DNS forwarding: VLANs → pfSense → Unbound (upstream)
- VLAN 50 has no DHCP — IPs are statically assigned to known worker MACs

## Key Design Decisions

- VLAN 50 (agent) has **no default route to VLANs 10, 20, or 30** — workers can reach the gateway for SOCKS5 egress but cannot initiate connections to management or server VLANs
- VLAN 10 (management) can reach all VLANs for administration; all other zones are default-deny toward 10
- Inter-VLAN routing is explicitly permitted per-rule, never by default