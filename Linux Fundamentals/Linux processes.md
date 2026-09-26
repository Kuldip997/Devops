
---

## What is a Process?

A running instance of a program. Every process gets a unique **PID**, and has a parent (**PPID**) that spawned it.

```bash
ps aux              # snapshot of all running processes
ps -ef                # similar, full-format
ps -u yourusername     # only your processes
```

Columns in `ps aux`:

```
USER  PID  %CPU  %MEM   VSZ   RSS TTY   STAT START   TIME COMMAND
```

## Live Monitoring — `top`, `htop`

```bash
top          # live view (q to quit)
htop           # nicer, colorized (sudo apt install htop)
```

Inside `top`:

- `P` — sort by CPU
- `M` — sort by memory
- `k` — kill a process (enter PID)
- `q` — quit

## Process States

|State|Meaning|
|---|---|
|`R`|Running or runnable|
|`S`|Sleeping (waiting for event)|
|`D`|Uninterruptible sleep (usually I/O)|
|`Z`|Zombie — finished but not cleaned up by parent|
|`T`|Stopped (paused)|

```bash
ps aux | awk '{print $8}' | sort | uniq -c    # count processes by state
```

## Foreground vs Background Jobs

```bash
sleep 100                 # foreground, blocks terminal
sleep 100 &                 # background, terminal free
jobs                          # list background jobs
fg %1                           # bring job 1 to foreground
bg %1                             # resume stopped job in background
Ctrl+Z                             # suspend current foreground job
Ctrl+C                             # kill current foreground job
```

## Killing Processes

```bash
kill 1234           # SIGTERM — graceful stop
kill -9 1234          # SIGKILL — force kill
kill -l                 # list all signal names
killall firefox           # kill by process name
pkill -f "python app.py"    # kill by matching command string
```

Common signals:

- `SIGTERM` (15) — polite stop, can be caught/ignored (default)
- `SIGKILL` (9) — immediate, cannot be caught
- `SIGHUP` (1) — hangup, often reloads configs
- `SIGSTOP`/`SIGCONT` — pause/resume

## Parent-Child Relationships

```bash
pstree                  # tree view of processes
pstree -p                 # include PIDs
ps -ef --forest             # ps in tree format
```

Every process traces back to `init`/`systemd` (PID 1) — first process started by the kernel at boot.

## Priority — `nice` and `renice`

Niceness ranges -20 (highest priority) to 19 (lowest). Default is 0.

```bash
nice -n 10 somecommand        # start with lower priority
renice -n 5 -p 1234              # change priority of running PID
```

Lower number = more CPU priority. Only root sets negative values.

## Background Services — `systemctl`

Long-running services (web servers, DBs) managed by **systemd**.

```bash
systemctl status ssh
sudo systemctl start ssh
sudo systemctl stop ssh
sudo systemctl restart ssh
sudo systemctl enable ssh      # auto-start on boot
sudo systemctl disable ssh       # don't auto-start
journalctl -u ssh                  # logs for that service
```

## Resource Usage Per Process

```bash
ps -p 1234 -o %cpu,%mem,cmd    # detailed usage for one PID
top -p 1234                       # monitor single PID live
```

## Finding What's Using a Port/File

```bash
lsof -i :8080          # process using port 8080
sudo netstat -tulpn        # listening ports + PIDs
sudo ss -tulpn                # modern replacement for netstat
```

---

## Practice Problems

- [ ] Find PID of current shell (`echo $$`), find its parent via `ps -ef`/`pstree -p`
- [ ] Start 3 `sleep` jobs in background, list with `jobs`, kill one via `kill %2`
- [ ] Search for zombie processes: `ps aux | grep 'Z'`
- [ ] Find top 5 memory-hungry processes: `ps aux --sort=-%mem | head -5`
- [ ] Check what's running on port 22 (SSH) using `lsof` or `ss`
- [ ] Open `top`, sort by memory (`M`), note top 3 processes
- [ ] Start `sleep 300 &`, find PID, kill gracefully (plain `kill`, not `-9`)
- [ ] Check status of `ssh` or `cron` via `systemctl status`
- [ ] Run `python3 -m http.server 8000 &`, find PID via `lsof -i :8000`, kill it

---

## Next Up

Log management (`journalctl`, `/var/log`)