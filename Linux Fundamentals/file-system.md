# Linux File System

#linux #filesystem #devops

---

## The Filesystem Hierarchy

Everything starts at `/` — one single tree, no drive letters like Windows. Disks/USBs get _mounted_ into this tree.

```bash
ls /
```

| Directory          | Purpose                                    |
| ------------------ | ------------------------------------------ |
| `/bin`, `/usr/bin` | essential user commands (ls, cp, bash)     |
| `/sbin`            | system admin commands (needs root)         |
| `/etc`             | system-wide config files                   |
| `/home`            | user home directories                      |
| `/root`            | home directory for root user               |
| `/var`             | variable data: logs, mail, caches          |
| `/tmp`             | temporary files, cleared on reboot         |
| `/dev`             | device files (disks, terminals, etc.)      |
| `/proc`, `/sys`    | virtual files exposing kernel/process info |
| `/mnt`, `/media`   | mount points for external filesystems      |
| `/opt`             | optional/third-party software              |
| `/lib`, `/usr/lib` | shared libraries                           |

## Absolute vs Relative Paths

```bash
cd /home/user/project     # absolute — always from /
cd project                 # relative — from current location
cd ./project                # same, explicit "here"
cd ../sibling                # relative, up then into sibling
```

## File Types

Everything in Linux is a "file" — check first char of `ls -l`:

| Symbol | Type             |
| ------ | ---------------- |
| `-`    | regular file     |
| `d`    | directory        |
| `l`    | symbolic link    |
| `c`    | character device |
| `b`    | block device     |
| `s`    | socket           |
| `p`    | named pipe       |

```bash
ls -l
file notes.txt      # identify type by content, not extension
```

## Links — Hard vs Symbolic

```bash
ln target.txt hardlink.txt        # hard link: same inode, same data
ln -s target.txt symlink.txt       # symlink: pointer to a path
```

- **Hard link** — same inode/data. Survives original deletion. No dirs, no cross-filesystem.
- **Symlink** — pointer to a path. Breaks if original is deleted/moved. Works on dirs, cross-filesystem.

```bash
ls -li target.txt hardlink.txt    # -i shows inode; matches for hardlink
```

## Inodes

Every file has an **inode**: metadata (permissions, owner, size, timestamps, data block pointers) — but NOT the filename. Filename = directory entry pointing to an inode.

```bash
ls -i file.txt      # inode number
stat file.txt         # full metadata
```

> Renaming = instant (only directory entry changes). Deleting a file only frees disk space once _all_ hard links to its inode are gone.

## Mounting

```bash
mount                # show mounted filesystems
df -h                  # disk usage, human-readable
lsblk                   # block devices as a tree
sudo mount /dev/sdb1 /mnt/usb
sudo umount /mnt/usb
```

Example `df -h`:

```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        50G   12G   36G  25% /
/dev/sda2       100G   80G   15G  85% /home
```

## Checking Size and Usage

```bash
du -sh foldername            # total size of a folder
du -h --max-depth=1 .          # size of each subfolder, one level
df -h                            # free/used space per filesystem
```

## Filesystem Types

- **ext4** — default on most distros, journaling
- **xfs** — common on RHEL/CentOS, good for large files
- **btrfs** — supports snapshots
- **vfat/ntfs** — Windows compatibility
- **tmpfs** — RAM-backed (e.g. `/tmp`)

```bash
mount | grep " / "     # filesystem type for root
```

## Special Files

```bash
cat /etc/fstab         # defines what mounts automatically at boot
cat /proc/cpuinfo        # live CPU info (virtual file)
cat /proc/meminfo         # live memory info
ls /dev                    # device files: sda, tty, null, zero
```

`/dev/null` = black hole, discards anything written to it:

```bash
some_command > /dev/null 2>&1
```

---

## Practice Problems

- [ ] Write absolute + relative path to reach `/etc/hosts` from home
- [ ] Create `original.txt`, a hard link `hard.txt`, and a symlink `soft.txt`; delete original and check which survives
- [ ] Run `stat` on a file — note inode, permissions, last modified
- [ ] Symlink a directory and `cd` into it
- [ ] Run `df -h` and `lsblk`; identify root partition
- [ ] Find home directory size broken down by subfolder (`du -h --max-depth=1`)
- [ ] Identify your home partition's filesystem type
- [ ] Redirect a command's output to `/dev/null` and confirm silence

---

