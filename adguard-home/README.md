# AdGuard Home

## Goal
The main goal deploying AdGuard Home is to block advertisements and tracking domains, improve DNS privacy, and prevent clients form bypassing local DNS filtering by using external DNS, DoT and DoH services.

---

## DNS Ports

- AdGuard Home: Port `53` - handles DNS queries from clients
- dnsmasq: Port `54` - kept for DHCP and local network services
- Unbound: Port `5335` - local recursive DNS resolver used by AdGuard Home

---

## Upstream DNS

AdGuard Home uses the local Unbound server running on `127.0.0.1:5335` as its upstream DNS server.

Unbound performs recursive DNS resolution instead of forwarding queries to an external resolver such as Quad9. DNSSEC validation and QNAME minimisation are also enabled in Unbound.

---

## DHCP Integration

DHCP is still handled by `dnsmasq`.

DHCP option 6 is configured for each active VLAN so clients receive the router interface address in their VLAN as the DNS server.

For example:
- Management: `192.168.10.1`
- MAIN: `192.168.20.1`
- PRIV: `192.168.30.1`
- Guest: `192.168.40.1`

This makes client devices use AdGuard Home automatically as their local DNS server.

---

## Firewall and DNS Enforcement

Firewall rules were configured to make clients use the local AdGuard Home DNS server.

For the Management, MAIN and Guest networks, direct access to external DNS servers on port `53` (TCP/UDP) is blocked. DNS over TLS (DoT) on port `853` is also blocked.

This prevents clients from bypassing AdGuard Home by manually configuring an external DNS server such as `8.8.8.8`.

Normal DNS queries to the router are still allowed and are handled by AdGuard Home.

---

## DoH Protection

The protection layers are used to reduce DNS over HTTPS (DoH) bypass.

AdGuard Home uses the HaGeZi Encrypted DNS list to block know domains used by public DoH servers.

In addition, banIP is running on the router with the DoH feed enabled. This feed blocks client connections to know DoH server IP address, so using a direct IP address does not easily bypass DNS filtering.

This setup block know DoH services, but it does not guarantee that every possible DoH server will be blocked.

---

## Verification

The configuration was tested from devices in the Management, MAIN and Guest networks.

DNS resolution through AdGuard Home was tested with:

```bash
nslookup openwrt.org
```

Direct access to an external DNS server was tested with:

```bash
nslookup openwrt.org 8.8.8.8
```

The query was blocked by the firewall.

DNS over TLS was tested with:

```bash
nc -vz 1.1.1.1 853
```

The connection was blocked.

DoH bypass using a direct server IP address was tested with:

```bash
curl --connect-timeout 5 \
--resolve dns.google:443:8.8.8.8 \
'https://dns.google/resolve?name=example.net&type=A'
```

The connection was blocked by banIP.

Unbound was also tested directly on port `5335`:

```bash
dig @127.0.0.1 -p 5335 openwrt.org A
```

DNSSEC validation was tested using a domain with an intentionally invalid DNSSEC signature:

```bash
dig @127.0.0.1 -p 5335 dnssec-failed.org A
```

Unbound returned `SERVFAIL`, confirming that invalid DNSSEC responses are rejected.

A correctly signed domain was also tested and the response contained the `ad` flag.

Finally, the router was rebooted and DNS resolution was tested again. AdGuard Home, Unbound and banIP started correctly after the reboot.

 
