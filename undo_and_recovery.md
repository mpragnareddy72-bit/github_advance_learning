# Git Advanced Learning — Undo & Recovery

## Part 1: Concepts

### 1. `git reset --soft`, `--mixed`, and `--hard`

`git reset` moves the current branch reference to another commit. Depending on the option, it also changes the staging area and working tree.

| Command | Commit history | Staging area | Working tree |
|---|---|---|---|
| `git reset --soft HEAD~1` | Moves back one commit | Keeps changes staged | Keeps changes |
| `git reset --mixed HEAD~1` | Moves back one commit | Unstages changes | Keeps changes |
| `git reset --hard HEAD~1` | Moves back one commit | Resets staging area | Resets tracked files to the target commit |

`--mixed` is the default option when no mode is specified.

#### Example

Suppose you committed unfinished README changes and product-search code, but the commit is only local.

- Use `--soft` if you want to undo the commit but keep the changes staged.
- Use `--mixed` if you want to undo the commit and keep the changes unstaged.
- Use `--hard` only when you intentionally want to discard changes to tracked files.

**Important:** Resetting can rewrite local history. Avoid resetting commits that teammates already depend on. A hard reset can discard work, so check `git status` and protect important changes first.

### 2. `git revert`

`git revert` creates a new commit that reverses the changes introduced by an earlier commit. It does not erase the original commit from history.

```bash
git revert <commit-hash>
```

Use it when a bad commit has already been pushed to a shared branch and teammates may have pulled it.

#### Reverting a merge commit with `-m 1`

A merge commit usually has two parents:

- Parent 1: the branch that was checked out when the merge was performed.
- Parent 2: the branch that was merged in.

```bash
git revert -m 1 <merge-commit-hash>
```

`-m 1` tells Git to treat the first parent as the mainline. Git creates a new commit that reverses the merge's changes relative to that parent.

Before running it, inspect the merge commit:

```bash
git show --no-patch --pretty=raw <merge-commit-hash>
```

Confirm which parent is the baseline you want to retain.

**Remember:** `-m 1` does not mean “revert one commit.” It means “use parent 1 as the mainline.”

### 3. Recovering commits and deleted branches with `git reflog`

The reflog records recent movements of local references such as `HEAD`. It can help locate commits after a reset, rebase, or branch deletion.

```bash
git reflog
```

Find the commit you want to recover. Then create a recovery branch:

```bash
git switch -c recovery/product-search <commit-hash>
```

This creates a new branch pointing to that commit, making it easier to inspect and recover your work.

The reflog is local and is not a permanent backup. Entries can expire, and unreachable commits can eventually be garbage-collected.

### 4. Undoing a bad rebase

#### If the rebase is still running

```bash
git rebase --abort
```

This cancels the in-progress rebase and attempts to restore the branch to its pre-rebase state.

#### If the rebase has already completed

Inspect the reflog:

```bash
git reflog
```

Find the previous branch position, then create a recovery branch:

```bash
git switch -c recovery/before-rebase <old-commit-hash>
```

Inspect and test the recovered work before deciding how to restore the original branch.

Do not assume that `HEAD@{1}` is always the correct recovery point. Read the reflog and identify the intended commit.

### 5. Recovering after a bad force push

A force push can replace the remote branch's visible history. Recovery may still be possible if the old commit exists in a local reflog, another clone, a backup, or another reference.

1. Stop making further history changes to the affected branch.
2. Find the old commit using `git reflog` or another clone that still has it.
3. Preserve it with a recovery branch.
4. Coordinate with teammates before changing the shared remote again.
5. Restore the intended history using a carefully reviewed push or another agreed recovery procedure.

If the old commit is known and the remote branch must be restored, a command such as the following may be needed:

```bash
git push --force-with-lease origin <branch-name>
```

This is not automatically safe just because it uses `--force-with-lease`. Verify the remote state and coordinate with collaborators first. If the old commit cannot be found locally, inspect other clones or backups.

### 6. `git revert` vs `git reset`

| Situation | Usually appropriate |
|---|---|
| Undo a local commit but keep its changes staged | `git reset --soft` |
| Undo a local commit and keep changes unstaged | `git reset --mixed` |
| Discard tracked working-tree changes intentionally | `git reset --hard` |
| Undo a pushed commit without rewriting shared history | `git revert` |
| Cancel an in-progress rebase | `git rebase --abort` |
| Locate a lost commit or previous branch position | `git reflog` |
| Preserve a recovered commit on a new branch | `git switch -c recovery/name <hash>` |

---

## Part 2: Practical Scenarios and Situations

### Scenario 1: Undo a local commit but keep changes unstaged

**Situation:** You committed unfinished README changes and product-search code. The commit has not been pushed. You want to undo the commit but keep all tracked changes in your working directory, unstaged.

**Question:** Which command should you use?

**Answer:**

```bash
git reset --mixed HEAD~1
```

**Why:** The branch moves back one commit, the changes are removed from the staging area, and the working-tree changes remain.

**Why not the other options?**

- `--soft`: keeps the changes staged.
- `--hard`: resets tracked files and can discard the changes.

### Scenario 2: A bad commit was already shared

**Situation:** You pushed a faulty product API commit. Your colleague has already pulled it.

**Question:** How should you undo it?

**Answer:**

```bash
git revert <bad-commit-hash>
```

**Why:** It creates a new commit that reverses the bad commit while preserving shared history.

**Why not reset?** Resetting and force-pushing would rewrite history that your colleague may already have.

### Scenario 3: A local feature branch was deleted

**Situation:** You deleted `feature/product-search`. Its commits are no longer visible in the normal log while you are on `main`.

**Question:** What should you inspect first?

**Answer:**

```bash
git reflog
```

Then recover the desired commit:

```bash
git switch -c feature/product-search <commit-hash>
```

**Why:** The reflog may still contain the branch's previous commit position. Creating a branch at that commit gives you a recoverable reference.

**Why not `git revert`?** Revert reverses changes; it does not recreate a deleted branch.

### Scenario 4: You started a rebase on the wrong branch

**Situation:** A rebase is still in progress, and you realize it was started on the wrong branch.

**Question:** What should you do?

**Answer:**

```bash
git rebase --abort
```

**Why:** It cancels the in-progress rebase.

If the rebase already completed, inspect `git reflog` to locate the previous state instead.

### Scenario 5: You need to undo a merge

**Situation:** You merged a feature branch into `main`. The merge introduced unwanted changes, and the merge commit has two parents.

**Question:** How can you revert the merge while retaining the first parent's baseline?

**Answer:**

```bash
git revert -m 1 <merge-commit-hash>
```

**Why:** `-m 1` selects parent 1 as the mainline. Git creates a new commit that reverses the merge's changes relative to that parent.

**Before running it:** Inspect the merge commit and confirm that parent 1 is the intended baseline.

### Scenario 6: A rebase completed and your changes appear lost

**Situation:** You completed a rebase, but your branch no longer contains the commit you expected.

**Question:** What should you do first?

**Answer:**

```bash
git reflog
```

Find the previous commit, then preserve it:

```bash
git switch -c recovery/before-rebase <old-commit-hash>
```

**Why:** The reflog can show where `HEAD` pointed before the rebase.

**Why not immediately reset the branch?** First preserve and inspect the recovered commit so you don't make the situation worse.

### Scenario 7: A bad force push affected a shared branch

**Situation:** You force-pushed the wrong history to GitHub. Your colleague may have work on the old branch.

**Question:** What should you do first?

**Answer:**

1. Stop further pushes.
2. Notify your colleague and coordinate.
3. Find the previous commit in a reflog, another clone, or a backup.
4. Preserve the old commit with a recovery branch.
5. Agree on the correct remote history before restoring it.

**Why:** Another force push without checking the current remote state could overwrite more work.

---

## Part 3: Quick Revision Questions

1. Which reset mode keeps changes staged?
2. Which reset mode keeps changes but unstages them?
3. Which command creates a new commit to undo a bad shared commit?
4. Which command helps find a lost commit?
5. Which command cancels a rebase that is still in progress?
6. What does `-m 1` mean when reverting a merge commit?
7. Why is rewriting shared history riskier than reverting a commit?

## Key Takeaway

- Use `reset` carefully for local history and staging changes.
- Use `revert` to undo shared commits without removing their history.
- Use `reflog` to find previous local reference positions.
- Use `rebase --abort` only for a rebase that is still in progress.
- Preserve recovered commits before attempting risky restoration.
