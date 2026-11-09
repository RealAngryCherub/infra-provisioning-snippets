# VM Deployment Notes — Proxmox

## Host Layout

| Host    | Role                       | Specs                     |
|---------|----------------------------|---------------------------|
| Host 1  | pfSense firewall/gateway   | 4C/8T, 8GB, 120GB SSD    |
| Host 2  | Windows Server + Linux VMs | 6C/12T, 16GB, 256GB SSD  |

## Typical VM Creation (qm)

Windows Server (Active Directory):

```bash
qm create 200 --memory 4096 --cores 2 --name AD-DC \
  --net0 bridge=vmbr0,tag=20 \
  --scsihw virtio-scsi-pci \
  --ide2 local:iso/WinServer2022.iso,media=cdrom
```

Linux worker node (Ubuntu Server 24.04):

```bash
qm create 210 --memory 2048 --cores 1 --name worker-node-1 \
  --net0 bridge=vmbr0,tag=50 \
  --ide2 local:iso/ubuntu-24.04-server.iso,media=cdrom
```

## Network Bridge Config

Each Proxmox host has a single bridge (`vmbr0`) that trunks to the pfSense-managed switch port. VLAN tagging happens at the VM level via `tag=` on the net0 line. This keeps the Proxmox host itself untagged (management on VLAN 10) while VMs land in their assigned zone.

```bash
# /etc/network/interfaces (on Proxmox host)
auto vmbr0
iface vmbr0 inet static
    address 10.x.x.x/24
    gateway 10.x.x.1
    bridge-ports <physical-nic>
    bridge-stp off
    bridge-fd 0
```

## Cloud-init Snippets

Worker VMs use cloud-init to set a low-privilege user on first boot with no sudo access by default:

```yaml
#cloud-config
users:
  - name: agent-runner
    sudo: false
    shell: /bin/bash
    ssh-authorized-keys:
      - <hermes-gateway-public-key>
runcmd:
  - apt update && apt upgrade -y
  - systemctl enable ssh
```