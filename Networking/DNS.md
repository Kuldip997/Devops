# DNS (Domain Name System)

#networking #dns #devops

---

## What is DNS?

Translates human-friendly domain names (`google.com`) into IP addresses (`142.250.195.78`) computers use to connect. The "phonebook of the internet."

```bash
ping google.com      # resolves the name to an IP under the hood first
```

## How a DNS Lookup Works

1. **Browser/OS cache** — already resolved recently? Use it.
2. **Recursive resolver** — your request goes here (ISP's or public, e.g. `8.8.8.8`). It does the work for you.
3. **Root nameserver** — resolver asks "who handles `.com`?" → gets TLD server address.
4. **TLD nameserver** — resolver asks "who handles `example.com`?" → gets authoritative nameserver.
5. **Authoritative nameserver** — resolver asks "what's the IP for `www.example.com`?" → gets the IP.
6. **Resolver caches and returns** the IP to your browser.

```
You -> Resolver -> Root -> TLD (.com) -> Authoritative NS -> IP returned
```

Usually milliseconds, heavily cached at every step.

## DNS Record Types

|Record|Purpose|Example|
|---|---|---|
|`A`|Domain -> IPv4|`example.com -> 93.184.216.34`|
|`AAAA`|Domain -> IPv6|`example.com -> 2606:2800:220:1::`|
|`CNAME`|Alias, name -> name|`www.example.com -> example.com`|
|`MX`|Mail server|`example.com -> mail.example.com`|
|`TXT`|Arbitrary text (SPF/DKIM/verification)|`"v=spf1 include:_spf.google.com ~all"`|
|`NS`|Authoritative nameservers for domain|`example.com -> ns1.example.com`|
|`SOA`|Start of Authority — admin info, zone timers|—|
|`PTR`|Reverse lookup, IP -> domain|`34.216.184.93.in-addr.arpa -> example.com`|
|`SRV`|Service + port pointer (SIP, XMPP)|—|

## Lookup Tools

```bash
nslookup example.com            # simple lookup, older tool
dig example.com                   # detailed, modern, preferred
dig example.com +short              # just the IP
dig example.com MX                    # specific record type
dig @8.8.8.8 example.com                # query a specific DNS server
host example.com                          # quick alternative
```

## Machine DNS Configuration

```bash
cat /etc/resolv.conf          # DNS servers your machine uses
systemd-resolve --status         # detailed resolver status (older)
resolvectl status                   # modern replacement
```

Typical `/etc/resolv.conf`:

```
nameserver 8.8.8.8
nameserver 1.1.1.1
```

Public resolvers: Google `8.8.8.8`/`8.8.4.4`, Cloudflare `1.1.1.1`/`1.0.0.1`, Quad9 `9.9.9.9`

## The Hosts File — Bypassing DNS

Linux checks `/etc/hosts` before DNS.

```bash
cat /etc/hosts
```

```
127.0.0.1    localhost
192.168.1.50  myserver.local
```

Useful for local testing — point a domain to a local dev server without touching real DNS.

```bash
sudo nano /etc/hosts        # add: 127.0.0.1  test.local
ping test.local                # resolves instantly, no DNS lookup
```

## Caching and TTL

Every record has a **TTL** (Time To Live) — how long resolvers may cache it.

```bash
dig example.com
# look for a number like 300 next to the A record = TTL in seconds
```

- Short TTL (60s) → fast propagation, more lookup traffic
- Long TTL (86400s = 1 day) → efficient, slower to propagate changes

This is why DNS changes can take minutes to 48 hours to "propagate" worldwide.

## Forward vs Reverse DNS

- **Forward** — name → IP (normal case)
- **Reverse** — IP → name, uses `PTR` record

```bash
dig -x 8.8.8.8          # reverse lookup
host 8.8.8.8               # also does reverse lookup
```

Used to verify mail servers (many reject mail from IPs with no valid reverse DNS).

## Authoritative vs Recursive vs Caching Servers

- **Authoritative nameserver** — source-of-truth records (Cloudflare, Route53, GoDaddy, etc.)
- **Recursive resolver** — does the multi-step lookup for you (ISP, `8.8.8.8`)
- **Caching server** — stores recent answers temporarily (resolvers, OS, browser)

```bash
dig example.com NS        # shows authoritative nameservers for a domain
```

## Troubleshooting DNS

```bash
dig example.com                     # is it resolving at all?
dig example.com +trace                # trace full path: root -> TLD -> authoritative
nslookup example.com 8.8.8.8            # test against a specific server
resolvectl flush-caches                   # clear local DNS cache
sudo systemd-resolve --flush-caches         # older command form
```

|Symptom|Likely cause|
|---|---|
|Works with `dig @8.8.8.8` but not plain `dig`|Local/ISP DNS server broken or slow|
|`NXDOMAIN`|Domain doesn't exist, or typo|
|`SERVFAIL`|DNS server error, or domain misconfigured|
|Resolves to wrong/old IP|Stale cache — check TTL, flush cache|
|Works on mobile data, not on your network|Router/ISP DNS is the problem|

## DNS and Security

- **DNSSEC** — cryptographically signs DNS records to prevent spoofing/tampering (`dig example.com +dnssec`)
- **DNS spoofing/cache poisoning** — attacker tricks a resolver into caching a fake IP
- **DoH / DoT** (DNS over HTTPS / TLS) — encrypts DNS queries so ISPs/networks can't see lookups in plain text

---

## Practice Problems

- [ ] `dig example.com` — identify IP and TTL
- [ ] `dig example.com MX` and `dig example.com TXT` — compare record types
- [ ] `dig example.com +short` vs `dig @1.1.1.1 example.com +short` — compare resolvers
- [ ] `cat /etc/resolv.conf` — check default DNS server
- [ ] Edit `/etc/hosts`: map `myapp.local` -> `127.0.0.1`, then `ping myapp.local`
- [ ] `dig -x 8.8.8.8` — reverse lookup, note the hostname returned
- [ ] `dig example.com NS` — identify who manages the domain's DNS
- [ ] `dig example.com +trace` — watch full root -> TLD -> authoritative resolution
- [ ] `resolvectl flush-caches` — clear local DNS cache

---

