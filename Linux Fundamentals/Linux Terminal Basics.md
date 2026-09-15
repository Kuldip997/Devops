# Linux Terminal Basics

#linux #terminal #cli

---

## Navigation
```bash
pwd        # current directory
ls / ls -l / ls -a / ls -la
cd foldername
cd ..      # up one level
cd ~       # home
cd -       # previous directory
```

## Create Files/Folders
```bash
mkdir myproject
mkdir -p a/b/c   # nested folders
touch notes.txt
```

## View File Content
```bash
cat file.txt      # print whole file
less file.txt      # scroll (q to quit)
head -n 5 file.txt   # first 5 lines
tail -n 5 file.txt   # last 5 lines
tail -f log.txt      # live watch
```

## Copy / Move / Delete
```bash
cp a.txt copy.txt
cp -r folder1 folder2
mv a.txt renamed.txt
rm a.txt
rm -r folder
rm -rf folder   # ⚠ no undo!
```

## Search
```bash
find . -name "*.txt"
grep "word" file.txt
grep -r "word" .
grep -i "word" file.txt   # case-insensitive
```

## Redirection & Pipes
```bash
echo "text" > file.txt     # overwrite
echo "text" >> file.txt     # append
ls -l | grep ".txt"          # pipe output
cat file.txt | wc -l          # count lines
```
> `|` sends one command's output into the next command as input.

## Permissions
```bash
ls -l file.txt          # view: -rwxr-xr--
chmod +x script.sh        # make executable
chmod 755 script.sh        # set exact permission
sudo chown user:group file.txt
```
Permission string = [file type][owner rwx][group rwx][other rwx]

## Processes
```bash
ps aux       # list processes
top           # live view (q to quit)
kill 1234      # stop process
kill -9 1234    # force kill
```

## Package Management (Debian/Ubuntu)
```bash
sudo apt update
sudo apt install curl
sudo apt remove curl
```

---

## Practice Problems
- [ ] Navigate to `/`, list contents, return home in one command
- [ ] Create `practice/` with `a.txt`, `b.txt`, `c.txt`
- [ ] Copy, rename, and delete files inside `practice/`
- [ ] Use `grep` to search a word inside a file
- [ ] Pipe `ls` output through `wc -l` to count files
- [ ] Create `project/src`, `project/docs`, `project/tests` in one `mkdir -p`
- [ ] Make a file executable and confirm with `ls -l`

---

