#networking #ip #devops

---

## What is an IP Address?

A unique numeric label assigned to a device on a network, so other devices know where to send data.

- **IPv4** — `192.168.1.10` (32-bit, 4 numbers 0–255 separated by dots)
- **IPv6** — `2001:0db8:85a3::8a2e:0370:7334` (128-bit, hex groups separated by colons)

IPv4 is still dominant day-to-day; IPv6 exists because IPv4's ~4.3 billion addresses ran out.

```bash
ip addr show        # modern way to see your IP addresses
ip a                  # shorthand
ifconfig                # older tool (sudo apt install net-tools)
```

## IPv4 Structure — Network vs Host Portion

An IPv4 address splits into a **network** portion (which network) and a **host** portion (which device on it). The **subnet mask** marks where the split happens.

```
IP:          192.168.1.10
Subnet mask: 255.255.255.0
```

In binary, `255` = `11111111` (network part), `0` = `00000000` (host part). So `192.168.1` is the network, `.10` is this device.

## CIDR Notation

Shorthand for the subnet mask — a slash followed by the number of network bits.

```
192.168.1.10/24
```

`/24` = first 24 bits (3 octets) are network, last 8 bits are host — same as `255.255.255.0`.

|CIDR|Subnet Mask|Host Addresses|
|---|---|---|
|`/8`|255.0.0.0|~16 million|
|`/16`|255.255.0.0|~65,000|
|`/24`|255.255.255.0|254|
|`/30`|255.255.255.252|2 (point-to-point links)|

## Public vs Private IP Addresses

- **Public IP** — routable on the internet, globally unique, assigned by ISP
- **Private IP** — only valid within a local network, reused everywhere, not internet-routable

Reserved private ranges:

```
10.0.0.0/8          -> 10.0.0.0 - 10.255.255.255
172.16.0.0/12        -> 172.16.0.0 - 172.31.255.255
192.168.0.0/16        -> 192.168.0.0 - 192.168.255.255
```

Routers use **NAT** (Network Address Translation) to let many private-IP devices share one public IP.

```bash
curl ifconfig.me        # your PUBLIC IP (as seen from the internet)
ip a                       # your PRIVATE IP (local network)
```

## Special / Reserved Addresses

```
127.0.0.1         -> localhost (loopback, always "this machine")
0.0.0.0            -> "all interfaces" (server listening on every NIC)
255.255.255.255     -> broadcast (send to everyone on local network)
169.254.x.x           -> APIPA, self-assigned when DHCP fails
```

```bash
ping 127.0.0.1        # pings yourself, never leaves the machine
```

## Static vs Dynamic IP Assignment

- **Static** — manually configured, never changes. Used for servers, printers.
- **Dynamic** — auto-assigned by **DHCP** (Dynamic Host Configuration Protocol), typically your router; can change over time.

```bash
ip route show         # see default gateway (router/DHCP server)
cat /etc/resolv.conf     # see DNS servers in use
```

## Checking and Configuring Interfaces

```bash
ip addr show                               # all interfaces and their IPs
ip link show                                 # interfaces + up/down state
sudo ip addr add 192.168.1.50/24 dev eth0      # temp static IP
sudo ip link set eth0 up                         # bring interface up
sudo ip link set eth0 down                         # bring interface down
```

> These changes are temporary (lost on reboot) unless saved to a persistent config (`/etc/netplan/*.yaml` on Ubuntu, `/etc/network/interfaces` on Debian, or via `nmcli`).

## Testing Connectivity

```bash
ping google.com              # check if a host is reachable
ping -c 4 8.8.8.8               # send only 4 packets, then stop
traceroute google.com             # path (hops) packets take
mtr google.com                      # ping + traceroute combined, live
```

## Ports — IP's Companion

An IP reaches a _machine_; a **port** reaches a specific _service_ on it. Written as `IP:port`.

```
192.168.1.10:22     -> SSH
192.168.1.10:80      -> HTTP
192.168.1.10:443      -> HTTPS
```

```bash
ss -tulpn          # listening ports on your machine
```

## DNS — Quick Preview

You type domain names, not IPs — **DNS** translates them behind the scenes.

```bash
nslookup google.com
dig google.com          # more detail (sudo apt install dnsutils)
```

Flow: domain name → DNS lookup → IP address → connection. (Covered fully in its own session.)

---

## Practice Problems

- [ ] Run `ip a`, note your private IP and subnet mask/CIDR
- [ ] Run `curl ifconfig.me`, compare to your private IP — note the difference (NAT)
- [ ] Ping `127.0.0.1`, then your private IP, then `google.com` — compare results
- [ ] Work out usable host addresses in a `/28` network (by hand, then verify)
- [ ] Run `ip route show`, identify your default gateway
- [ ] Run `traceroute`/`mtr` to a public site, note the first hop (should be your router)
- [ ] If your IP is `192.168.1.10/24`, how many other hosts can share that network?

---

## Next Up

Subnetting math (splitting networks into smaller ones), or DNS in depth