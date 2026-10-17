# Incorect system time after reboot

### Problem
After rebooting the router, I noticed that the system date was incorrect. The router was still showing an older date.

This cloud cause problems with services that depend on correct system time, such as DNSSEC and HTTPS.

### Solution:

The router was using NTP servers such as:

```text
0.openwrt.pool.ntp.org
1.openwrt.pool.ntp.org
2.openwrt.pool.ntp.org
3.openwrt.pool.ntp.org
```

During startup, DNS was not fully available yet because AdGuard Home was trying to use Unbound before Unbound was ready.
Because of this, the router could not resolve the NTP server domain names and the system time was not synchronized.
Temporary solution
I manually synchronized the system time using a Cloudflare NTP server by its IP address:
```bash
ntpd -q -p 162.159.200.1
```

Then I verified the system date:
```bash
date
```

The date was corrected successfully.
The router currently still displays time in UTC. I plan to configure the correct local timezone later in the project.

