# VLAN and SSID Configuration

## Goal

The goal of this part of the project was to divide the network into four separate Wi-Fi network to each of them.

I am aware that creating a separate Wi-Fi network for each VLAN increases the wireless attack surface. However, this router has only one LAN port, so I decided to use this solution.

Each Wi-Fi network uses WPA3-SAE encryption and a strong password. Two of the SSIDs are hidden. I did this mainly to reduce number of visible network names and to make these network less obvious to people nearby. I am aware that hiding an SSID is not a security measure, because hidden networks cat still be discovered easily.

Each VLAN has a different purpose and will later have its own firewall rules. At this stage, I only allowed the traffic required for DHCP and DNS. I will not describe these firewall rules here because the firewall configuration will be documented separately.

---

## Network Design

| VLAN | Name | Subnet | Purpose |
|------|------|--------|---------|
| 10 | Managment | 192.168.10.0/24 | Router administration |
| 20 | MAIN | 192.168.20.0/24 | Daily usage |
| 30 | PRIV | 192.168.30.0/24 | Private use of the internet |
| 40 | Guest | 192.168.40.0 | Guest devices |

---

## Bridge VLAN Filtering

I enabled VLAN filtering to separate the network based on VLAN IDs and added VLAN 10, 20, 30 and 40.

The LAN port was assigned to VLAN 20 as untagged/PVID, so a device connected via Ethernet is placed in the MAIN network.

Each VLAN was separately assigned to its corresponding SSID.

![br-lan_and_vlans](/screenshots/br-lan_and_vlans.png)

![bridge_vlan_filtering](/screenshots/br-lan_and_vlans.png)

## VLAN 10 - Managment

### Purpose

This VLAN is used only for router administration. Only my laptop will have access to this VLAN, and I may also allow my phone in the future.

### Interface

- Interface: `Managment`
- Device: `br-lan.10`
- Router IP: `192.168.10.1`
- Subnet: `192.168.10.0/24`

### DHCP

- DHCP: Enabled
- IPv6: Disabled
- Range: `192.168.10.100 - 192.168.10.200`
- Lease time: `12h`

### Wi-Fi / SSID

The Management VLAN is assigned to a dedicated Wi-Fi network.

- SSID `Net-Admin`
- Mode: Access Point
- Security: WPA3-SAE
- Hidden SSID: Yes
- Client Isolation: Yes

### Access
This network is mainly used to access LuCi and SSH. It does not have access to other VLANs. More details about the access rules will be documented separately in the firewall section.



