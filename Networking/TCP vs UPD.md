
#networking #tcp #udp #devops

---

## What a Transport Protocol Does

IP moves packets between machines but guarantees nothing about delivery/order. TCP and UDP sit on top of IP and define _how_ data actually moves between applications.

```
Application (HTTP, DNS, SSH...)
      |
Transport (TCP or UDP)
      |
Network (IP)
      |
Link (Ethernet/WiFi)
```

## TCP — Transmission Control Protocol

**Connection-oriented** and **reliable**. Handshake before any data moves.

**3-way handshake:**

```
Client -> SYN       -> Server      "I want to connect"
Client <- SYN-ACK   <- Server      "OK, acknowledge, do you?"
Client -> ACK       -> Server      "Acknowledged, connection established"
```

Closing has its own teardown (FIN/ACK exchange, ~4 steps).

**Guarantees:**

- Ordering — packets arrive in sequence, even via different routes
- Reliability — lost packets detected and retransmitted
- Flow control — receiver can tell sender to slow down
- Error checking — checksums catch corruption

```bash
ss -t                      # show TCP connections
ss -t state established      # only established TCP connections
```

**Used for:** HTTP/HTTPS, SSH, FTP, email (SMTP/IMAP/POP3), databases — anywhere losing/reordering data breaks things.

## UDP — User Datagram Protocol

**Connectionless** and **unreliable by design**. No handshake, no acknowledgment, no ordering guarantee.

```
Client -> datagram -> Server      (sent and forgotten)
```

**Characteristics:**

- Faster — no handshake overhead
- No ordering guarantee
- No retransmission — lost packet is just gone
- Lower overhead — 8-byte header vs TCP's 20+

```bash
ss -u          # show UDP sockets
```

**Used for:** DNS queries, video/audio streaming, VoIP, online gaming, DHCP — where speed matters more than perfection.

## Side-by-Side Comparison

|Feature|TCP|UDP|
|---|---|---|
|Connection|Connection-oriented (handshake)|Connectionless|
|Reliability|Guaranteed, retransmits lost packets|No guarantee|
|Ordering|Guaranteed in-order|No guarantee|
|Speed|Slower (handshake/acks/retransmits)|Faster|
|Header size|20+ bytes|8 bytes|
|Flow control|Yes|No|
|Error checking|Yes, with recovery|Checksum only, no recovery|
|Use case|Accuracy > speed|Speed > accuracy|

## Why DNS Uses Both

DNS mostly uses **UDP** (port 53) for quick request/response. Falls back to **TCP** (also port 53) when the response is too large for one UDP packet (zone transfers, large record sets).

```bash
dig example.com +tcp        # force DNS query over TCP
```

## Why Video Calls Use UDP, Not TCP

If TCP dropped a packet mid-call, it would pause and wait to retransmit before showing anything newer — freezing the picture. UDP just drops that frame and keeps playing live — smoother despite occasional glitches.

> **TCP prioritizes correctness over timeliness. UDP prioritizes timeliness over correctness.**

## Observing the Handshake

```bash
sudo tcpdump -i any port 80 -c 20      # capture first 20 packets on port 80
```

Load a page (`curl http://example.com`) in another terminal and watch for `[S]` (SYN), `[S.]` (SYN-ACK), `[.]` (ACK) flags — the handshake happening live.

## Checking Protocol Usage for a Port

```bash
ss -tulpn | grep :53      # DNS - likely shows both tcp and udp lines
ss -tulpn | grep :80        # HTTP - tcp only
ss -tulpn | grep :443         # HTTPS - tcp only
```

## Sequence Numbers and Acknowledgments

Every byte in a TCP stream is numbered:

```
Client sends bytes 1-1000    -> seq=1
Server acknowledges             -> ack=1001 ("got everything up to 1000, send 1001 next")
```

If bytes 1001-2000 are lost, server keeps acknowledging 1001 until client resends — this is retransmission, driven by acknowledgment gaps, no separate error messages needed.

## Common Misconceptions

- **"UDP is always worse than TCP"** — different tool for different jobs; for real-time data, imperfect-but-now often beats perfect-but-late.
- **"A port is either TCP or UDP"** — a port number is just a number; the same number (e.g. 53) can have an independent TCP listener and UDP listener simultaneously.
- **"TCP never loses data"** — TCP detects loss and retransmits, but a severed connection (cable cut, server crash) can still lose in-transit data; TCP only guarantees what you _do_ receive is complete and ordered.

---

## Practice Problems

- [ ] `ss -t state established` — view active TCP connections
- [ ] `ss -u` — see what's using UDP right now
- [ ] `ss -tulpn | grep :53` — confirm DNS shows both tcp and udp
- [ ] `dig example.com +tcp` — force DNS lookup over TCP, compare to normal `dig`
- [ ] `sudo tcpdump -i any port 80 -c 10` + `curl -s http://example.com > /dev/null` in another terminal — find SYN/SYN-ACK/ACK in the capture
- [ ] Think through: why would an FPS game prefer UDP for player position updates but TCP for chat messages?

