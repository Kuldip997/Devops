
#git #github #devops

---

## How a Merge Works

Git finds the **merge base** (most recent common ancestor), compares each branch's changes against it, and combines them. Non-overlapping changes merge automatically; overlapping ones conflict.

```
          base
            |
main:   A - B - E
             \
feature:      C - D
```

```bash
git merge-base main feature        # shows the common ancestor commit
```

## Fast-Forward vs Merge Commit

```bash
git merge feature               # fast-forward if possible, else merge commit
git merge --no-ff feature         # always create a merge commit
git merge --ff-only feature         # refuse unless fast-forward is possible
```

- `--no-ff` keeps a visible record that a feature branch existed
- `--ff-only` guarantees linear history, fails loudly otherwise

```bash
git config --global merge.ff false          # make --no-ff the default
git config --global pull.ff only              # make `git pull` fast-forward only
```

## Squash Merge

Applies all of a branch's commits as **one staged change**. No merge commit; individual commits are not preserved.

```bash
git switch main
git merge --squash feature
git commit -m "Add login feature"         # you commit it yourself
```

```
feature: A - B - C - D - E          main after squash:  A - B - [CDE as one commit]
```

- Good for messy WIP branches; keeps `main` clean
- Trade-off: lose detailed history; Git doesn't see the branch as merged

```bash
git branch -d feature             # refuses: "not fully merged"
git branch -D feature               # needed after a squash merge
```

## Merge vs Rebase

||Merge|Rebase|
|---|---|---|
|History|Preserves true history, adds merge commits|Rewrites into a straight line|
|Commits|Originals unchanged|Recreated (new hashes)|
|Safe on shared branches|Yes|No, never rebase pushed/shared work|
|Conflicts|Resolved once|May resolve per replayed commit|

```bash
git switch feature
git rebase main                     # replay feature's commits on top of main
git switch main
git merge feature                     # now a clean fast-forward
```

```
Before:  main: A - B - E           feature: A - B - C - D
Rebase:  feature: A - B - E - C' - D'
Merge:   main fast-forwards to D'
```

> Rule of thumb: rebase your own local work to tidy up before sharing; merge to integrate shared branches.

```bash
git rebase --abort                   # undo a rebase in progress
git rebase --continue                  # after fixing a conflict and `git add`
git reflog                               # find pre-rebase state if needed
```

## Resolving Conflicts

```bash
git status                          # lists "both modified" (unmerged) files
git diff                              # shows conflicting hunks
```

Take one side wholesale:

```bash
git checkout --ours file.txt          # keep current branch's version
git checkout --theirs file.txt          # keep incoming branch's version
git add file.txt
```

> During a **merge**, "ours" = branch you're on, "theirs" = branch being merged in. During a **rebase** they're swapped.

Strategy options (apply to all conflicts in the merge):

```bash
git merge -X ours feature              # prefer our side on conflicting hunks
git merge -X theirs feature              # prefer theirs on conflicting hunks
```

> `-X ours` (option) differs from `-s ours` (strategy that discards the other branch's changes entirely). Be careful with both.

**Show the common ancestor in markers (diff3 style):**

```bash
git config --global merge.conflictStyle zdiff3
```

```
<<<<<<< HEAD
our version
||||||| base
original version
=======
their version
>>>>>>> feature
```

**Visual tools:**

```bash
git mergetool
git config --global merge.tool vimdiff    # or meld, vscode, kdiff3
```

## Conflict Resolution Checklist

```
1. git merge feature                 -> CONFLICT
2. git status                          -> find unmerged files
3. open each file, resolve <<<< ==== >>>> sections
4. remove ALL conflict markers          (grep -rn "<<<<<<<" .)
5. run tests / build                      (don't skip)
6. git add <resolved files>
7. git commit                              (or git merge --continue)
```

```bash
grep -rn "<<<<<<<\|>>>>>>>" .          # check no markers left behind
git merge --abort                        # bail out any time before committing
```

## Cherry-Pick — One Commit, Not the Whole Branch

```bash
git log --oneline feature                 # find the commit hash
git switch main
git cherry-pick a1b2c3d                     # copy that commit onto main
git cherry-pick a1b2c3d e4f5g6h               # multiple commits
git cherry-pick --abort                         # cancel if it conflicts
```

Typical use: a hotfix on `main` that also needs to go into a release branch. The copy gets a **new hash**.

## Undoing a Merge

**Not yet committed (conflict state):**

```bash
git merge --abort
```

**Merged locally, not pushed:**

```bash
git reset --hard HEAD~1                  # move branch back one commit (discards merge)
git reset --hard ORIG_HEAD                 # Git remembers pre-merge position in ORIG_HEAD
```

**Already pushed/shared** — don't rewrite history; revert instead:

```bash
git revert -m 1 <merge-commit-hash>       # -m 1 = keep first parent (main's side)
```

`-m 1` is required for merge commits (two parents): tells Git which line is the "mainline".

> ⚠ After reverting a merge, re-merging the same branch won't bring the changes back; revert the revert first.

## Merging in GitHub Pull Requests (Preview)

|GitHub option|Git equivalent|
|---|---|
|Create a merge commit|`git merge --no-ff`|
|Squash and merge|`git merge --squash`|
|Rebase and merge|rebase + fast-forward|

Teams pick one as a convention.

## Useful Merge Inspection Commands

```bash
git log --merges --oneline                 # only merge commits
git log --no-merges --oneline                # hide merge commits
git log --first-parent --oneline                # follow only the mainline
git show <merge-commit>                          # a merge's combined diff
git diff main...feature                            # changes on feature since it diverged (3 dots)
git branch --merged main                             # branches already merged into main
```

---

## Practice Problems

- [ ] `git merge-base main feature` on two diverged branches; find it in `git log --graph --oneline --all`
- [ ] Merge the same branch with `--ff` and `--no-ff`; compare graphs
- [ ] 3 commits on a branch, `git merge --squash` into `main`; note `-d` refuses to delete it
- [ ] Create a conflict, resolve by hand; redo with `git checkout --theirs`
- [ ] Set `merge.conflictStyle zdiff3`, create a conflict, read the three-section markers
- [ ] Rebase a feature branch onto `main`, fast-forward merge, compare hashes before/after
- [ ] Cherry-pick one commit from a branch onto `main`
- [ ] Merge a branch, undo with `git reset --hard ORIG_HEAD`; merge again, undo with `git revert -m 1`
