
#git #github #devops

---

## What is Git?

A **distributed version control system** — tracks file changes over time, lets multiple people work on the same project, keeps full history to revert to. "Distributed" means every clone has the _entire_ history, not just the latest snapshot.

```bash
git --version        # check it's installed
sudo apt install git    # install if missing (Debian/Ubuntu)
```

## First-Time Setup

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main      # use "main" for new repos
git config --list                                   # see current settings
```

## Creating a Repository

```bash
mkdir myproject && cd myproject
git init                    # turns this folder into a git repository
ls -la                        # notice the hidden .git folder — history lives here
```

Everything git tracks lives inside `.git/`. Delete it and you lose version history (files on disk stay as-is).

## The Three Areas of Git

```
Working Directory  ->  Staging Area (Index)  ->  Repository (committed history)
   (your files)         (git add)                  (git commit)
```

- **Working directory** — actual files on disk
- **Staging area** — holding zone for what goes into the next commit
- **Repository** — permanent, committed history

```bash
echo "hello world" > notes.txt     # create a file (working directory)
git status                           # tracked/untracked/staged state
```

## Staging and Committing

```bash
git add notes.txt          # stage a specific file
git add .                    # stage everything changed/new in this directory
git status                     # check what's staged vs not

git commit -m "Add notes.txt"    # commit staged changes with a message
git log                            # commit history
git log --oneline                    # compact one-line-per-commit view
```

A commit is a permanent snapshot: unique hash, author, timestamp, message, and pointer to the previous commit (forming the history chain).

## Checking What Changed

```bash
git diff                # changes in working directory, not yet staged
git diff --staged         # changes staged but not yet committed
git show <commit-hash>      # full details of a specific commit
```

## Ignoring Files — `.gitignore`

Some files shouldn't be tracked: logs, build artifacts, secrets, dependency folders.

```bash
touch .gitignore
echo "*.log" >> .gitignore
echo "node_modules/" >> .gitignore
echo ".env" >> .gitignore
```

Git skips anything matching these patterns in `git status` and `git add .`.

## Undoing Things (the safe ones)

```bash
git restore notes.txt           # discard unstaged changes in working directory
git restore --staged notes.txt    # unstage a file (keeps the changes, just unstages)
git commit --amend -m "new message"    # edit the most recent commit's message
```

> ⚠ `git restore` (without `--staged`) permanently discards uncommitted changes — no undo for that specific action.

## Viewing History in Detail

```bash
git log --graph --oneline --all      # visual graph of commits/branches
git log -p                              # full diff for each commit
git log --author="Your Name"              # filter by author
git log --since="2 days ago"                # filter by time
```

## Removing and Renaming Tracked Files

```bash
git rm filename            # delete file + stage the deletion
git mv old.txt new.txt        # rename + stage the rename, in one step
```

Plain `rm` + `git add .` works too, but `git rm`/`git mv` do both steps together.

---

## Practice Problems

- [ ] `git init` a new folder, confirm name/email configured globally
- [ ] Create a file, check `git status`, `git add` it, check `git status` again — note wording change
- [ ] `git commit -m "..."`, view with `git log --oneline`
- [ ] Edit the file, run `git diff`, stage it, run `git diff --staged` — compare outputs
- [ ] Create `.gitignore`, add a pattern, create a matching file, confirm it's ignored
- [ ] Make an uncommitted edit, `git restore` it, confirm reverted
- [ ] Create and commit a second file, `git rm` it, commit the deletion
- [ ] `git log --graph --oneline --all` — view the shape of your history

