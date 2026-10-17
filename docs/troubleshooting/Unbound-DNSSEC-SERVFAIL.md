# Problem: Unbound DNSSEC returned SERVFAIL after reboot

### Problem

After rebooting the router, DNS stopped working correctly.

DNS queries through AdGuard Home returned `SERVFAIL`, even though the router still had Internet access by IP address.

### Symptoms

- `ping 8.8.8.8` worked correctly
- `nslookup openwrt.org` returned `SERVFAIL`
- `apk update` did not work correctly
- direct queries to Unbound also returned `SERVFAIL`

Example:

```bash
dig @127.0.0.1 -p 5335 openwrt.org A
```

Result:

```text
status: SERVFAIL
```

### Diagnostics

At first, I noticed that the router system time was incorrect.

The router showed October 7 instead of October 15.

Because Unbound was using DNSSEC validation, incorrect system time could cause problems with DNSSEC signature validation.

After manually correcting the system time, the problem still remained.

I checked the Unbound logs:

```bash
logread | grep -i unbound | tail -n 30
```

The logs showed:

```text
failed to prime trust anchor -- DNSKEY rrset is not secure
```

I then checked the Unbound trust anchor file:

```bash
head -n 20 /var/lib/unbound/root.key
```

The file contained two root keys:

```text
20326 → VALID
38696 → ADDPEND
```

The key `38696` was not yet trusted.

I also discovered that OpenWrt was using another trust anchor file after restarting Unbound:

```text
/etc/unbound/root.key
```

This file still contained the older trust anchor state, where key `38696` was still marked as:

```text
ADDPEND
```

### Cause

The problem was related to the DNSSEC root trust anchor state.

Unbound had an old trust anchor state where the newer root key `38696` was not yet trusted.

After restarting Unbound, OpenWrt reused the older `/etc/unbound/root.key` file.

Because of this, Unbound could not correctly initialize DNSSEC validation and returned `SERVFAIL`.

The incorrect system time also made the problem harder to diagnose and could interfere with DNSSEC validation.

### Solution

First, I temporarily disabled DNSSEC validation to restore normal DNS resolution:

```bash
uci set unbound.ub_main.validator='0'
uci commit unbound
service unbound restart
```

After DNS started working again, I installed the `unbound-anchor` tool:

```bash
apk add unbound-anchor
```

I first updated the runtime trust anchor:

```bash
unbound-anchor -a /var/lib/unbound/root.key
```

However, after restarting Unbound, the problem returned because OpenWrt was still using the older source file:

```text
/etc/unbound/root.key
```

I created a backup:

```bash
cp /etc/unbound/root.key /etc/unbound/root.key.bak
```

Then I updated the correct trust anchor file:

```bash
unbound-anchor -a /etc/unbound/root.key
```

After that, the file contained both current root trust anchors.

I then enabled DNSSEC again:

```bash
uci set unbound.ub_main.validator='1'
uci commit unbound
service unbound restart
```

### Verification

A correctly signed domain was tested with:

```bash
dig @127.0.0.1 -p 5335 openwrt.org A
```

Result:

```text
status: NOERROR
```

A domain with intentionally broken DNSSEC was tested with:

```bash
dig @127.0.0.1 -p 5335 dnssec-failed.org A
```

Result:

```text
status: SERVFAIL
```

This confirmed that DNSSEC validation was working correctly again.

I also cleared the DNS cache in AdGuard Home because it had cached earlier `SERVFAIL` responses and continued returning them even after Unbound was fixed.

### What I learned

- DNSSEC depends on correct system time
- a broken or outdated trust anchor can cause `SERVFAIL`
- Unbound can use more than one `root.key` file
- fixing Unbound does not automatically clear cached errors in AdGuard Home
- DNS problems should be tested layer by layer
- testing AdGuard Home and Unbound separately makes troubleshooting much easier

### Note

This problem was fully diagnosed and solved with help from ChatGPT.

At my current level of experience, I would not have been able to identify the DNSSEC trust anchor problem on my own.

I used ChatGPT to guide the troubleshooting process step by step, while I executed and verified every command on the router.

This troubleshooting process helped me better understand how system time, DNSSEC, Unbound trust anchors, AdGuard Home caching, and recursive DNS can interact during a failure.
