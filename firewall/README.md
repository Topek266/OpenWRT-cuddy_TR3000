# Firewall

## Goal

This firewall configuration was created to separate all VLANs from each other and limit comunication between them. For example, devices in the Guest network should not be able to access devices in the MAIN VLAN.

I use a rule of blocking traffic that is not needed and allowing only the communications that are required. This helps prevent someone who gains access to one VLAN from easily reaching other network segments.

Each VLAN has its own firewall rules based on its purpose.

---

## Firewall Zones

| Zone       | Input  | Output | Forward | Purpose               |
| ---------- | ------ | ------ | ------- | --------------------- |
| MANAGEMENT | REJECT | ACCEPT | REJECT  | Router administration |
| MAIN       | REJECT | ACCEPT | REJECT  | Daily trusted network |
| PRIV       | ...    | ...    | ...     | VPN-only network      |
| GUEST      | REJECT | ACCEPT | REJECT  | Guest Internet access |
| WAN        | ...    | ...    | ...     | LTE Internet          |

---

## VLAN 10 - MANAGEMENT

### Zone Policy
- input:   `REJECT`
- output:  `ACCEPT`
- forward: `REJECT`

### Allowed Services
- DHCP
- DNS
- HTTPS
- SSH only from `192.168.10.10`
- ICMP to all router interfaces

### SSH Restriction
- Administration laptop has reserved IP `192.168.10.10`
- SSH access is allowed only from this address
- SSH uses a custom non-default port

### Verification

I verified the firewall rules from the administration laptop.

```bash
ip addr
nslookup openwrt.org 192.168.10.1
nc -vz 192.168.10.1 80
nc -vz 192.168.10.1 443
nc -vz 192.168.10.1 53
nc -vzu 192.168.10.1 67
ssh -p <custom-port> root@192.168.10.1
ping -c 4 192.168.10.1
ping -c 4 192.168.20.1
ping -c 4 192.168.30.1
ping -c 4 192.168.40.1
ping -c 4 1.1.1.1
```

- The laptop received the correct reserved IP address: `192.168.10.10`
- DNS resolved domain names correctly
- HTTPS on port `443` was accessible
- DNS on port `53` was working
- DHCP was working correctly
- HTTP on port `80` was blocked
- SSH connection worked correctly from `192.168.10.10`
- ICMP to the router VLAN interfaces worked correctly
- Internet access worked correctly and the router could download packages and updates

## VLAN 20 - MAIN

### Zone Policy
- input:   `REJECT`
- output:  `ACCEPT`
- forward: `REJECT`

Forwarding:
MAIN --> wan: `ALLOW`

### Allowed Services
- DHCP
- DNS
- ICMP to MAIN router interface

### Verification

```bash
ip addr
nslookup openwrt.org 192.168.10.1
nc -vz 192.168.20.1 80
nc -vz 192.168.20.1 443
nc -vz 192.168.20.1 53
nc -vzu 192.168.20.1 67
nc -vz 192.168.10.10 443
ssh -p <custom-port> root@192.168.10.1
ping -c 4 192.168.10.1
ping -c 4 192.168.20.1
ping -c 4 192.168.30.1
ping -c 4 192.168.40.1
ping 1.1.1.1
```

- DHCP assigned an IP address from the configured range
- DNS resolved domain names correctly
- DHCP worked correctly
- Internet access worked correctly
- DNS on port `53` was accessible
- HTTP and HTTPS access to the router were blocked
- SSH access to the router were blocked
- ICMP worked only to `192.168.20.1` and the internet test address `1.1.1.1`

## VLAN 30 -Privacy

> Note: I do not have the VPN service configured yet, so the VPN and kill switch rules are not tested yet.

### Zone Policy
- input:   `REJECT`
- output:  `ACCEPT`
- forward: `REJECT`

### Allowed Services
- DHCP
- DNS 

### Verification

```bash
ip addr
nslookup openwrt.org 192.168.10.1
nc -vz 192.168.10.1 80
nc -vz 192.168.10.1 443
nc -vz 192.168.10.1 53
nc -vzu 192.168.10.1 67
nc -vz 192.168.10.1 443
ssh -p <custom-port> root@192.168.10.1
ping -c 4 192.168.10.1
ping -c 4 192.168.20.1
ping -c 4 192.168.30.1
ping -c 4 192.168.40.1
ping 1.1.1.1
```

- DHCP assigned an IP address from the configured range
- DNS resolved domain names correctly
- DHCP worked correctly
- DNS on port `53` was accessible
- HTTP and HTTPS access to the router were blocked
- SSH access to the router were blocked

## VLAN 40 - Guest

### Zone Policy
- Input:   `REJECT`
- Output:  `ACCEPT`
- Forward: `REJECT`

Forwarding --> Guest: `ALLOW`

### Allowed Services
- DHCP
- DNS
- Internet access

### Verification

```bash
ip addr
nslookup openwrt.org 192.168.10.1
nc -vz 192.168.10.1 80
nc -vz 192.168.10.1 443
nc -vz 192.168.10.1 53
nc -vzu 192.168.10.1 67
nc -vz 192.168.10.1 443
ssh -p <custom-port> root@192.168.10.1
ping -c 4 192.168.10.1
ping -c 4 192.168.20.1
ping -c 4 192.168.30.1
ping -c 4 192.168.40.1
ping 1.1.1.1
```

- DHCP assigned an IP address from the configured range
- DNS resolved domain names correctly
- DHCP worked correctly
- DNS on port `53` was accessible
- HTTP and HTTPS access to the router were blocked
- SSH access to the router were blocked

## WAN / LTE

### Zone Policy

The WAN zone uses the following policy
- Input:   `REJECT`
- Output:  `ACCEPT`
- Forward: `REJECT`
- IPv4 masquerading: `enabled`

The LTE interface belongs to the WAN firewall zone

### Default OpenWRT Rules

OpenWRT created several default WAN rules for DHCP renewall, ICMP, multicast, IPv6 and IPsec-related traffic. These rules were left unchanged.

### Access from VLANs

- MANAGEMENT --> WAN: no general forwarding
- MAIN --> WAN: allowed
- PRIV --> WAN: not configured
- Guest --> WAN: allowed

The MANAGEMENT VLAN has only a separate ICMP rule for diagnostic purposes.

