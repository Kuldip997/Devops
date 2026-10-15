
#git #github #devops

---

## What is a Branch?

An independent line of development. Work on a feature/fix without touching stable code. A branch is just a lightweight **pointer to a commit**, so creating one is instant.

```
main:     A --- B --- C
                       \
feature:                D --- E
```

```bash
git branch            # list local branches (* = current)
git branch -a           # list local + remote branches
```

## HEAD

Pointer to your current position, usually the tip of the current branch. Committing moves the branch forward, and `HEAD` with it.

```bash
git log --oneline --decorate      # shows HEAD -> main next to latest commit
```

## Creating and Switching Branches

```bash
git branch feature-login            # create (stay on current)
git switch feature-login              # move to that branch
git switch -c feature-login             # create AND switch (most common)
git switch -                              # back to previous branch
```

Older equivalent: `git checkout feature-login` / `git checkout -b feature-login`. `switch` is the modern, clearer command.

## Branches Isolate Your Work

```bash
git switch -c feature-a
echo "feature A work" > a.txt
git add a.txt && git commit -m "Add a.txt on feature-a"

git switch main
ls                          # a.txt is gone, it only exists on feature-a
git switch feature-a
ls                            # it's back
```

Git rewrites your working directory to match the branch you switch to.

## Merging

Switch to the branch that should **receive** changes, then merge the other in.

```bash
git switch main
git merge feature-a
```

**Fast-forward** — `main` hasn't moved since branching; Git just slides the pointer forward, no new commit.

```
Before:  main: A --- B          feature: A --- B --- C --- D
After:   main: A --- B --- C --- D   (pointer moved)
```

**Three-way merge** — both branches have new commits; Git creates a **merge commit** with two parents.

```
main:     A --- B --- E ------- M
                \             /
feature:         C --- D ----
```

```bash
git merge --no-ff feature-a       # force a merge commit even if fast-forward is possible
```

## Merge Conflicts

Happens when two branches changed **the same lines** of the same file.

Create one on purpose:

```bash
git switch -c conflict-demo
echo "line from conflict-demo" > shared.txt
git add . && git commit -m "Add shared.txt on conflict-demo"

git switch main
echo "line from main" > shared.txt
git add . && git commit -m "Add shared.txt on main"

git merge conflict-demo          # CONFLICT
```

Git marks the file:

```
<<<<<<< HEAD
line from main
=======
line from conflict-demo
>>>>>>> conflict-demo
```

- `<<<<<<<` to `=======` = **current branch's** version
- `=======` to `>>>>>>>` = **incoming branch's** version

Resolve:

```bash
# 1. Edit the file: keep one side or combine, delete the markers
# 2. Mark resolved
git add shared.txt
# 3. Finish the merge
git commit

git merge --abort                    # or cancel the merge entirely
git status                             # lists "both modified" files during a conflict
```

## Deleting Branches

```bash
git branch -d feature-a          # delete a merged branch (safe, refuses if unmerged)
git branch -D feature-a            # force delete even if unmerged (work is lost)
```

Deleting a merged branch only removes the pointer; commits stay in `main`'s history.

## Renaming and Comparing

```bash
git branch -m old-name new-name       # rename
git branch --merged                      # branches already merged into current
git branch --no-merged                     # branches with unmerged work
git diff main..feature-a                     # difference between two branches
git log main..feature-a --oneline              # commits on feature-a that main lacks
```

## Stashing — Park Unfinished Work

```bash
git stash                       # shelve uncommitted changes
git stash list                    # see saved stashes
git stash pop                       # reapply latest stash and remove it
git stash apply                       # reapply but keep it in the list
git stash drop                          # delete the latest stash
```

## Common Branching Workflow

```
1. git switch main && git pull           # start from up-to-date main
2. git switch -c feature/my-change         # branch for one piece of work
3. ...edit, git add, git commit (repeat)...
4. git switch main && git merge feature/my-change
5. git branch -d feature/my-change           # clean up
```

Naming conventions: `feature/login-page`, `bugfix/null-pointer`, `hotfix/payment-crash`, `release/1.2.0`

## Rebase (Preview)

Replays your commits on top of another branch for a **linear history** instead of a merge commit.

```bash
git switch feature-a
git rebase main
```

```
Before:  main: A - B - E        feature: A - B - C - D
After:   main: A - B - E        feature: A - B - E - C' - D'   (C and D rewritten)
```

> ⚠ Never rebase commits you've already pushed and shared — rebase rewrites history.

---

## Practice Problems

- [ ] `git switch -c feature-a`, add a file, commit it
- [ ] `git switch main`, confirm file is gone, switch back, confirm it's back
- [ ] Merge `feature-a` into `main`, check `git log --oneline --graph`, note if fast-forward
- [ ] Create two branches editing the **same line**, merge both, resolve the conflict by hand
- [ ] `git branch --merged`, delete merged branches with `-d`
- [ ] Make an uncommitted edit, `git stash`, switch branches, switch back, `git stash pop`
- [ ] `git log main..feature-x --oneline` on an unmerged branch
- [ ] Compare your `git log --graph --oneline --all` output with what you expect

