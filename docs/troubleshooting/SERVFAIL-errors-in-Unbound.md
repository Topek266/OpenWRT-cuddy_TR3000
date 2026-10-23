## Problem: Intermittent SERVFAIL errors in Unbound

After configuring AdGuard Home with Unbound as the local DNS resolver, I started experiencing intermittent DNS resolution problems.

The DNS architecture was:
`Client → AdGuard Home :53 → Unbound :5335 → Internet`

Symptoms included:
- nslookup openwrt.org returning SERVFAIL,
- direct queries to Unbound also returning SERVFAIL:
```bash
dig @127.0.0.1 -p 5335 openwrt.org A
```

- AdGuard Home logging timeouts when communicating with the upstream server 127.0.0.1:5335,
- the problem sometimes disappearing on its own and returning later,
- router restarts not always triggering the issue immediately.
During troubleshooting, I confirmed that:
- the router had working Internet connectivity by IP,
- direct queries to root DNS servers worked,
- Unbound was using both IPv4 and IPv6:
do-ip4: yes
do-ip6: yes

My LTE uplink is effectively used as an IPv4 connection, but Unbound was still trying to perform recursive DNS resolution over IPv6 as well. This may have caused delays and timeouts during DNS recursion.

I changed Unbound to use IPv4 only:
```bash
uci set unbound.ub_main.protocol='ip4_only'
uci commit unbound
service unbound restart
```

After the change, the effective configuration became:
do-ip4: yes
do-ip6: no

After applying this configuration:
- direct dig queries returned NOERROR,
- nslookup worked correctly,
- after fully disconnecting the router from power and booting it again, DNS still worked correctly.
At this point, the most likely cause of the intermittent SERVFAIL errors was Unbound attempting to use IPv6 in an environment where the LTE uplink was effectively operating over IPv4.

This will still need longer-term observation to confirm that the issue is fully resolved.
