
#networking #ports #devops

---

## What is a Port?

An IP address reaches a _machine_; a port reaches the right _application_ on it. Range: **0–65535**.

```
192.168.1.10:22     -> SSH
192.168.1.10:80      -> HTTP
192.168.1.10:443      -> HTTPS
```

IP = street address, port = apartment/door number.

## Port Ranges

|Range|Name|Use|
|---|---|---|
|0–1023|Well-known/system ports|Reserved for standard services (needs root to bind)|
|1024–49151|Registered ports|Assigned to specific apps by IANA|
|49152–65535|Dynamic/private/ephemeral ports|Temporary, used for outgoing client connections|

```bash
cat /etc/services | head -20    # known port assignments on your system
```

## Common Ports

|Port|Service|
|---|---|
|20/21|FTP (data/control)|
|22|SSH|
|23|Telnet (insecure, avoid)|
|25|SMTP (sending email)|
|53|DNS|
|67/68|DHCP|
|80|HTTP|
|110|POP3 (email retrieval)|
|143|IMAP (email retrieval)|
|443|HTTPS|
|3306|MySQL|
|5432|PostgreSQL|
|6379|Redis|
|8080|HTTP alternate (dev servers)|
|27017|MongoDB|

## TCP vs UDP Ports

Ports belong to a protocol — same number can be used independently by both.

- **TCP** — connection-based (handshake: SYN, SYN-ACK, ACK), reliable, ordered. Web, SSH, email, file transfer.
- **UDP** — connectionless, fast, no delivery guarantee. DNS queries, video streaming, VoIP, gaming.

```bash
ss -tulpn          # t=tcp, u=udp, l=listening, p=process, n=numeric
```

## Checking What's Listening

```bash
ss -tulpn                       # modern tool
sudo netstat -tulpn               # older tool, same purpose
sudo lsof -i :80                    # what's using port 80
sudo lsof -i -P -n | grep LISTEN      # all listening processes, numeric ports
```

Example output:

```
Netid  State   Local Address:Port   Process
tcp    LISTEN  0.0.0.0:22            sshd
tcp    LISTEN  127.0.0.1:5432         postgres
```

`0.0.0.0` = listening on all interfaces (reachable from anywhere). `127.0.0.1` = only reachable from the same machine.

## Testing if a Port is Open/Reachable

```bash
nc -zv hostname 22              # check if port 22 is open (netcat)
nc -zv hostname 20-100             # scan a range of ports
telnet hostname 80                   # manually test a TCP port
curl -v telnet://hostname:80            # alternative using curl
nmap hostname                             # full port scan (sudo apt install nmap)
nmap -p 22,80,443 hostname                  # scan specific ports only
```

## Opening a Port Yourself — Quick Test Server

```bash
python3 -m http.server 8000         # web server on port 8000
nc -lvp 9000                          # netcat listens on port 9000
```

In another terminal:

```bash
curl localhost:8000           # directory listing response
nc localhost 9000               # connects; typed text appears on the other side
```

## Firewalls and Ports

A firewall controls which ports accept traffic, and from where.

```bash
sudo ufw status                     # Ubuntu's simple firewall
sudo ufw allow 22                     # allow SSH
sudo ufw allow 8080/tcp                 # allow specific port + protocol
sudo ufw deny 23                          # block telnet
sudo ufw enable                             # turn on firewall

sudo iptables -L -n -v              # lower-level, more powerful firewall tool
```

> A port can be "open" (service listening) but still unreachable if the firewall blocks it — two separate layers.

## NAT and Port Forwarding

A home router has one public IP but many devices behind it. **Port forwarding** tells the router: traffic hitting public IP on port X goes to this internal device.

```
Internet -> Router (public IP, port 8080) -> forwarded -> 192.168.1.50:80 (your server)
```

How you host something at home and make it reachable from the internet.

## Common Connection Errors

|Error|Meaning|
|---|---|
|`Connection refused`|Nothing listening on that port (service down or wrong port)|
|`Connection timed out`|Firewall dropping packets, wrong IP, or host unreachable|
|`No route to host`|Network-level issue, host unreachable at all|
|Port shows open in `nmap` but connection fails|App-level issue — service accepted then crashed/misbehaved|

---

## Practice Problems

- [ ] `cat /etc/services | head -20` — browse known port-to-service mappings
- [ ] `ss -tulpn` — list listening services, note TCP vs UDP
- [ ] Find one service on `0.0.0.0` and one on `127.0.0.1` — explain the difference
- [ ] `nc -zv localhost 22` — confirm SSH port is open
- [ ] `python3 -m http.server 8000 &`, then `curl localhost:8000` — confirm response
- [ ] `nc -zv localhost 9999` — see what a closed port looks like
- [ ] `sudo ufw status` or `sudo iptables -L -n -v` — check firewall rules
- [ ] `nmap localhost` — see every open port on your own machine

---

