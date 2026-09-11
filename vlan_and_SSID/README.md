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
| 10 | Management | 192.168.10.0/24 | Router administration |
| 20 | MAIN | 192.168.20.0/24 | Daily usage |
| 30 | PRIV | 192.168.30.0/24 | Private use of the internet |
| 40 | Guest | 192.168.40.0/24 | Guest devices |

---

## Bridge VLAN Filtering

I enabled VLAN filtering to separate the network based on VLAN IDs and added VLAN 10, 20, 30 and 40.

The LAN port was assigned to VLAN 20 as untagged/PVID, so a device connected via Ethernet is placed in the MAIN network.

Each VLAN was separately assigned to its corresponding SSID.

![bridge_vlan_filtering](/screenshots/br-lan_and_vlans.png)
![br-lan_and_vlans](/screenshots/br-lan_and_vlans.png)

---

## VLAN 10 - Management

### Purpose

This VLAN is used only for router administration. Only my laptop will have access to this VLAN, and I may also allow my phone in the future.

### Interface

- Interface: `Management`
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
This network is mainly used to access LuCI and SSH. It does not have access to other VLANs. More details about the access rules will be documented separately in the firewall section.

## VLAN 20 - Main

### Purpose
This VLAN is used for daily network access and my trusted devices, such as my laptop and phone. Devices can connect through Wi-Fi, and VLAN 20 is also used for physical LAN port.

### Interface

- Interface: `Main`
- Device: `br-lan.20`
- Router IP: `192.168.20.1`
- Subnet: `192.168.20.0/24`

### DHCP

- DHCP: Enabled
- IPv6: Disabled
- Range: `192.168.20.100 - 192.168.20.200`
- Lease time: `12h`

### Wi-Fi / SSID

The Main VLAN is assigned to a dedicated Wi-Fi network.

- SSID `Net-Main`
- Mode: Access Point
- Security: WPA3-SAE
- Hidden SSID: No
- Client Isolation: No

### Acces
This network has internet access, but it does access to other VLANs by default. More details about the access rules will be documented in the firewall section.

## VLAN 30 - Privacy

### Purpose
This VLAN is used for traffic that I want to separate from my daily network usage. It will use Mullvad VPN with kill switch to hide my public IP address from the services I access.

### Interface

- Interface: `Privacy`
- Device: `br-lan.30`
- Router IP: `192.168.30.1`
- Subnet: `192.168.30.0/24`

### DHCP

- DHCP: Enabled
- IPv6: Disabled
- Range: `192.168.30.100 - 192.168.30.200`
- Lease time: `12h`

### Wi-Fi / SSID

The Privacy VLAN is assigned to a dedicated Wi-Fi network.

- SSID `Net-Privacy`
- Mode: Access Point
- Security: WPA3-SAE
- Hidden SSID: Yes
- Client Isolation: Yes

### Access
This network will have internet access only through Mullvad VPN. If the VPN connection fails, the kill switch will block internet access. This VLAN will not have access to the other VLANs or router management services. More details about the access rules will be documented in the firewall section.

## VLAN 40 - Guest

### Purpose
This VLAN is used for friends and other guests who need internet access through my router. The network is separated from my trusted devices and other VLANs, so guests cannot access devices or traffic in my internal network.

### Interface

- Interface: `Guest`
- Device: `br-lan.40`
- Router IP: `192.168.40.1`
- Subnet: `192.168.40.0/24`

### DHCP

- DHCP: Enabled
- IPv6: Disabled
- Range: `192.168.40.100 - 192.168.40.200`
- Lease time: `12h`

### Wi-Fi / SSID

The Guest VLAN is assigned to a dedicated Wi-Fi network.

- SSID `Net-Guest`
- Mode: Access Point
- Security: WPA3-SAE
- Hidden SSID: No
- Client Isolation: Yes

### Access
This VLAN has internet access only. It does not have access to other VLANs or router management services. More details about the access rules will be documented in the firewall section.

---

## Final Verification

- Each VLAN receives addresses from the correct DHCP subnet.
- Each SSID is assigned to the correct VLAN.
- VLANs are isolated from each other.
- VLAN 10 provides access to LuCI and SSH.
- VLAN 20 has Internet access through Wi-Fi and Ethernet.
- Guest devices have Internet access without access to internal networks.
