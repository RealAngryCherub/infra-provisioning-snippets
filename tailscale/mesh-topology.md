# Tailscale Mesh Topology

## Layout

```
┌─────────────────┐     ┌─────────────────┐
│  Home Lab       │     │  Debian VPS     │
│  (pfSense, PVE) │────▶│  (Caddy, Docker)│
│  100.x.x.x      │     │  100.x.x.x      │
└────────┬────────┘     └────────┬────────┘
         │                      │
         │    Tailscale Mesh    │
         │   (zero-trust mTLS) │
         │                      │
┌────────▼──────────────────────▼─────────┐
│  Client devices (phone, laptop, etc.)   │
│  Tailscale app → direct mesh connection │
└─────────────────────────────────────────┘
```

## IP Assignment

- Tailscale assigns a `100.x.x.x/24` address to each node in the tailnet
- No public IPs needed — everything routes over the mesh
- The VPS is configured as a **subnet router** (--advertise-routes) for the home-lab LAN subnets

## ACL / Access Control

- VPS: accessible from all tailnet nodes (public services, Caddy proxied)
- Home-lab gateway (pfSense): accessible only from authorized admin devices
- Worker laptops: accessible only from the Hermes gateway for task delegation

## Tailscale Serve (exposing loopback services)

Services bound to `127.0.0.1` on the VPS are exposed to the tailnet via:

```bash
tailscale serve --bg --https=8445 8110
```

This gives a `https://<node>.tail729ec.ts.net:8445/` URL accessible **only** to tailnet-authorized devices.

## Key Design Decisions

- **No port forwarding on the ISP router** — Tailscale replaces port forwarding entirely
- **No public DNS records** for internal services — only Tailscale DNS names
- Subnet routing means home-lab services (like Wazuh dashboard, Proxmox web UI) are reachable via tailnet without exposing them publicly