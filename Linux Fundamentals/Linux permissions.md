# Linux Permissions

#linux #permissions #devops

---

## The Permission String

```bash
ls -l
```

```
-rwxr-xr--  1 user group  4096 Jan 1 script.sh
```

|Part|Meaning|
|---|---|
|`-`|file type (`-`=file, `d`=dir, `l`=symlink)|
|`rwx`|**owner** permissions|
|`r-x`|**group** permissions|
|`r--`|**others** permissions|

- `r` = read (view file / list directory)
- `w` = write (edit file / add-remove files in directory)
- `x` = execute (run file / enter directory with `cd`)
- `-` = permission denied

## Numeric (Octal) Permissions

`r=4, w=2, x=1` — sum per triplet.

|Combo|Value|
|---|---|
|`rwx`|7|
|`rw-`|6|
|`r-x`|5|
|`r--`|4|
|`---`|0|

`rwxr-xr--` = **754**

Common values:

- `644` → owner rw, group/others read-only (typical files)
- `755` → owner full, group/others read+execute (scripts/dirs)
- `600` → owner rw only, nobody else (SSH keys, private files)
- `777` → everyone full access (avoid — security risk)

## Changing Permissions — `chmod`

**Numeric mode:**

```bash
chmod 755 script.sh
chmod 644 notes.txt
chmod 600 id_rsa
```

**Symbolic mode:**

```bash
chmod +x script.sh              # add execute for everyone
chmod u+x script.sh               # add execute for owner only
chmod g-w file.txt                  # remove write from group
chmod o-r file.txt                    # remove read from others
chmod a+r file.txt                      # add read for all
chmod u=rwx,g=rx,o=r file.txt             # set explicitly per category
```

`u`=user(owner), `g`=group, `o`=others, `a`=all

## Changing Ownership — `chown`, `chgrp`

```bash
sudo chown newuser file.txt
sudo chown newuser:newgroup file.txt
sudo chgrp newgroup file.txt
sudo chown -R user:group folder/     # recursive
```

## Directory Permissions

- `r` = can list contents (`ls`)
- `w` = can create/delete files inside
- `x` = can `cd` into it / access files inside

> Gotcha: `r` without `x` — see filenames but can't `cd` in or open files. `x` without `r` — access known filenames but can't list contents.

```bash
mkdir testdir
chmod 600 testdir      # no execute
cd testdir               # FAILS — permission denied
```

## Special Permissions

**SUID** — file runs with owner's privileges (e.g. `/usr/bin/passwd`)

```bash
chmod u+s file
ls -l /usr/bin/passwd    # 's' in owner slot: -rwsr-xr-x
```

**SGID** — file: runs with group's privileges. Directory: new files inherit dir's group.

```bash
chmod g+s directory
ls -ld directory           # 's' in group slot: drwxr-sr-x
```

**Sticky bit** — only file owner (or root) can delete/rename, even with shared write access (e.g. `/tmp`)

```bash
chmod +t directory
ls -ld /tmp          # 't' at end: drwxrwxrwt
```

Numeric form: add 4th digit — `4755` (SUID), `2755` (SGID), `1755` (sticky).

## Default Permissions — `umask`

New files/dirs subtract umask from max (files `666`, dirs `777`).

```bash
umask            # e.g. 0022
```

umask `022` → files become `644`, dirs become `755`

## Checking Access — `sudo`, `su`

```bash
sudo command       # run one command as root
sudo -i               # root shell
su username             # switch user
whoami                    # current user
id                          # uid, gid, groups
```

---

## Practice Problems

- [ ] Work out numeric value for `rw-rw-r--` before checking with `ls -l`
- [ ] Create `deploy.sh`: owner full, group read+execute, others nothing (symbolic mode)
- [ ] Check ownership of a file in `/var/log`
- [ ] Create a dir, `chmod 600`, try `cd` into it, observe failure, fix with `chmod 700`
- [ ] `hello.sh` with `echo "hello"` — run it, fails, add execute, run again
- [ ] Create shared folder with SGID, add a file, check inherited group
- [ ] Find SUID files: `find / -perm -4000 2>/dev/null`
- [ ] Change umask temporarily (`umask 077`), create file, check permissions, reset

---
