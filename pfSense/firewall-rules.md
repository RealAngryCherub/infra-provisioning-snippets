# Firewall Rules — Default-Deny Per Zone

## Philosophy

Every VLAN starts with **no traffic permitted** to any other VLAN. Rules are added explicitly per interface pair. The default rule at the bottom of each tab is **Deny All**.

## Interface Groups

```
LAN (VLAN 10/20/30/40/50/60/80) — all internal interfaces
WAN — internet
OPT1 — OpenVPN tunnel interface
```

## Rule Order (top to bottom)

### WAN IN

| # | Action | Proto | Source       | Dest Port   | Description            |
|---|--------|-------|--------------|-------------|------------------------|
| 1 | Pass   | TCP   | *            | 443 (HTTPS) | Caddy web traffic      |
| 2 | Pass   | UDP   | *            | 1194 (OVPN) | OpenVPN (optional)     |
| 3 | Deny   | *     | *            | *           | Block everything else  |

### LAN IN (VLAN 30 — User Devices)

| # | Action | Proto | Source       | Dest / Port      | Desc                     |
|---|--------|-------|--------------|------------------|--------------------------|
| 1 | Pass   | *     | VLAN 30 net  | VLAN 20 (server) | User → AD, DNS, Wazuh   |
| 2 | Pass   | *     | VLAN 30 net  | VLAN 80          | User → DMZ (proxy)       |
| 3 | Pass   | TCP   | VLAN 30 net  | WAN (any)        | General internet         |
| 4 | Deny   | *     | VLAN 30 net  | VLAN 10          | No user access to mgmt   |
| 5 | Deny   | *     | VLAN 30 net  | VLAN 50          | No user access to agents |

### LAN IN (VLAN 50 — Agent Workers)

| # | Action | Proto | Source        | Dest / Port         | Desc                          |
|---|--------|-------|---------------|---------------------|-------------------------------|
| 1 | Pass   | TCP   | VLAN 50 net   | WAN (any)           | Outbound via SOCKS5 gateway   |
| 2 | Deny   | *     | VLAN 50 net   | VLAN 10/20/30/40/80 | No LAN access from workers    |
| 3 | Deny   | *     | VLAN 50 net   | *                   | Default deny toward everything|

### OpenVPN (OPT1 IN)

| # | Action | Proto | Source         | Dest / Port  | Desc                       |
|---|--------|-------|----------------|--------------|----------------------------|
| 1 | Pass   | *     | VPN pool (10.x) | VLAN 10/20  | Remote admin → mgmt + svrs |
| 2 | Deny   | *     | VPN pool       | VLAN 30/40/50/80 | No VPN to user/guest zones |

## Notes

- Rule numbers are logical order, not literal pfSense rule IDs
- Floating/states rules are enabled (established connections return traffic)
- These rules are enforced on the **LAN** tab (pfSense per-interface firewall), not the floating tab