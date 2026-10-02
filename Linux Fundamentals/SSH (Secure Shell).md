
---

## What is SSH?

Encrypted remote login to another machine. Replaced insecure tools like `telnet` and `rsh`.

- Default port: **22**
- **Client** = machine you connect _from_ (`ssh` command)
- **Server** = machine you connect _to_ (runs `sshd` daemon)

```bash
ssh -V                            # check client installed
sudo apt install openssh-server     # install server (Debian/Ubuntu)
systemctl status ssh                 # check if server is running
```

## Basic Connection

```bash
ssh username@hostname
ssh username@192.168.1.10
ssh -p 2222 username@hostname        # custom port
ssh username@hostname "ls -l"          # run one remote command, then exit
```

First connection shows a fingerprint prompt. Typing `yes` saves the server's fingerprint in `~/.ssh/known_hosts`. If it changes later, SSH warns you (possible man-in-the-middle attack or reinstalled server).

## Password vs Key-Based Auth

Key-based auth is more secure and the DevOps standard.

- **Private key** (`id_ed25519`) — stays on your machine, never share
- **Public key** (`id_ed25519.pub`) — placed on servers you want to access

Server verifies you hold the matching private key. No secret crosses the network.

## Generating a Key Pair

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

- File location: Enter for default (`~/.ssh/id_ed25519`)
- Passphrase: optional but recommended (encrypts private key on disk)

```bash
ls ~/.ssh
# id_ed25519       <- private key (keep secret!)
# id_ed25519.pub   <- public key (safe to share)
```

> `ed25519` is the modern recommended type. Older alternative: `-t rsa -b 4096`.

## Copying Public Key to a Server

```bash
ssh-copy-id username@hostname
```

Appends your public key to `~/.ssh/authorized_keys` on the server.

Manual method:

```bash
cat ~/.ssh/id_ed25519.pub | ssh username@hostname "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

Required permissions (SSH refuses keys if too open):

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_ed25519
```

## SSH Config File

`~/.ssh/config`:

```
Host myserver
    HostName 192.168.1.10
    User ubuntu
    Port 22
    IdentityFile ~/.ssh/id_ed25519

Host prod
    HostName prod.example.com
    User deploy
    Port 2222
```

Then simply:

```bash
ssh prod
```

## Copying Files — `scp` and `rsync`

```bash
scp file.txt user@host:/home/user/              # local -> remote
scp user@host:/home/user/file.txt .               # remote -> local
scp -r myfolder user@host:/home/user/              # copy directory

rsync -avz myfolder/ user@host:/home/user/myfolder/   # only sends changes
```

`rsync` flags: `-a` archive (keeps permissions), `-v` verbose, `-z` compress. Preferred for large/repeated transfers.

## SSH Agent

Enter passphrase once per session:

```bash
eval "$(ssh-agent -s)"         # start agent
ssh-add ~/.ssh/id_ed25519        # add key
ssh-add -l                         # list loaded keys
```

## Port Forwarding (Tunnels)

```bash
ssh -L 8080:localhost:80 user@host       # local: localhost:8080 -> remote port 80
ssh -R 9000:localhost:3000 user@host      # remote: expose local 3000 on server's 9000
ssh -D 1080 user@host                       # dynamic: SOCKS proxy
```

## Server Hardening — `/etc/ssh/sshd_config`

```
PermitRootLogin no              # no direct root login
PasswordAuthentication no        # keys only
PubkeyAuthentication yes
Port 2222                          # optional: change default port
AllowUsers alice bob                # restrict login to these users
```

Restart after editing:

```bash
sudo systemctl restart ssh
```

> ⚠ Confirm key login works in a second terminal BEFORE disabling password login, or you can lock yourself out.

## Troubleshooting

```bash
ssh -v user@host       # verbose (-vvv for more)
```

|Error|Usual cause|
|---|---|
|`Permission denied (publickey)`|Public key not in `authorized_keys`, or wrong file permissions|
|`Connection refused`|SSH server not running, or wrong port|
|`Connection timed out`|Firewall blocking, wrong IP, or host down|
|`REMOTE HOST IDENTIFICATION HAS CHANGED`|Server key changed; fix with `ssh-keygen -R hostname`|

---

## Advanced

### ProxyJump / Bastion Hosts

Access private servers through a jump host instead of connecting directly.

```bash
ssh -J bastion-user@bastion-host target-user@target-host
```

In `~/.ssh/config`:

```
Host bastion
    HostName bastion.example.com
    User jumpuser

Host prod-db
    HostName 10.0.5.12
    User deploy
    ProxyJump bastion
```

`ssh prod-db` now tunnels through the bastion automatically. Standard pattern for private cloud VPCs.

### Connection Multiplexing (ControlMaster)

Reuse one TCP connection for multiple sessions to the same host — much faster on repeated connects.

```
Host *
    ControlMaster auto
    ControlPath ~/.ssh/sockets/%r@%h-%p
    ControlPersist 600
```

### SSH Certificates (instead of raw keys)

At scale, distributing public keys to every server doesn't work. Sign user keys with a CA instead:

```bash
ssh-keygen -s ca_key -I user_id -n username -V +1d user_key.pub
```

Servers trust the CA (`TrustedUserCAKeys` in `sshd_config`), not individual keys. Enables short-lived certs instead of permanent keys sitting in `authorized_keys` forever.

### Agent Forwarding — and its risk

```bash
ssh -A user@host
```

Forwards your local SSH agent so you can hop onward to a third host using local keys, without copying them.

> ⚠ If the intermediate host is compromised, root there can hijack your forwarded agent and use your key while you're connected. Prefer `ProxyJump` over agent forwarding where possible.

### Ciphers, MACs, KexAlgorithms

```bash
ssh -Q cipher      # list supported ciphers
ssh -Q mac           # list supported MACs
ssh -Q kex             # list supported key exchange algorithms
```

Harden a server by restricting to modern algorithms in `sshd_config`:

```
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com
KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org
MACs hmac-sha2-512-etm@openssh.com
```

Useful for compliance scans (PCI-DSS, CIS benchmarks).

### Two-Factor Authentication

```bash
sudo apt install libpam-google-authenticator
google-authenticator        # sets up TOTP for the user
```

In `sshd_config`:

```
AuthenticationMethods publickey,keyboard-interactive
ChallengeResponseAuthentication yes
```

Login now needs both the private key AND a rotating TOTP code.

### `Match` Blocks — Conditional Config

Apply different rules per user, group, or IP range in `sshd_config`:

```
Match User backup-svc
    PasswordAuthentication no
    ForceCommand /usr/local/bin/backup-only.sh

Match Address 192.168.1.0/24
    PasswordAuthentication yes
```

`ForceCommand` restricts a key to only ever run one specific command — common for CI/CD or backup service accounts that should never get a shell.

### Restricting Keys in `authorized_keys`

Lock down an individual key's power without touching `sshd_config`:

```
command="/usr/bin/rsync --server --sender -vlogDtpre.iLsfx . /backup/",no-port-forwarding,no-X11-forwarding,no-agent-forwarding ssh-ed25519 AAAA...
```

This key can only run that rsync command, nothing else — how you give a CI pipeline deploy access without a real shell.

### SSH over Non-Standard Transport

```bash
ssh -o ProxyCommand="nc -X 5 -x proxyhost:1080 %h %p" user@host        # via SOCKS proxy
ssh -o ProxyCommand="cloudflared access ssh --hostname %h" user@host     # via Cloudflare Tunnel
```

Used when a target has no public IP/port 22 exposed — common in zero-trust setups.

### Auditing and Intrusion Detection

```bash
last                         # login history
lastb                          # failed login attempts
journalctl -u ssh --since today    # today's SSH daemon logs
sudo fail2ban-client status sshd      # banned IPs, if fail2ban installed
```

`fail2ban` auto-bans IPs after repeated failed attempts — standard on internet-facing servers.

### Protocol-Level Notes

- SSH-2 fully replaced SSH-1, which has known cryptographic weaknesses and shouldn't exist anywhere.
- Host keys use trust-on-first-use (TOFU) by default — no built-in CA for host identity unless configured (see certificates above). This is why the first-connection fingerprint prompt matters.
- `~/.ssh/known_hosts` entries can be hashed (`HashKnownHosts yes`) so a stolen laptop doesn't reveal which hosts you connect to.

---

## Practice Problems

- [ ] Generate an `ed25519` key pair with a passphrase
- [ ] View public key with `cat` (one long line starting `ssh-ed25519`)
- [ ] Check `ls -l ~/.ssh`: private key `600`, folder `700`
- [ ] Install `openssh-server` locally, run `ssh-copy-id localhost`, then `ssh localhost`
- [ ] Add a `local` entry in `~/.ssh/config`, connect with `ssh local`
- [ ] `scp` a file to `localhost:/tmp/` and verify
- [ ] Run `ssh -v localhost` and identify the auth method used
- [ ] Start `python3 -m http.server 8000 &`, tunnel with `ssh -L 9090:localhost:8000 localhost`, open `localhost:9090`
- [ ] Read `/etc/ssh/sshd_config`, find `Port` and `PermitRootLogin`
- [ ] Set up a `ProxyJump` entry in `~/.ssh/config` through a second local VM/container
- [ ] Run `ssh -Q cipher` and `ssh -Q kex`, note which algorithms are listed
- [ ] Add a restricted `command=` key to `authorized_keys` and confirm it can't run anything else
- [ ] Check `last` and `lastb` on your system for login history

---

