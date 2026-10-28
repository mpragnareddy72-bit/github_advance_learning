# Git Branching & Collaboration Practice Scenarios

## How to Use This Document

Try answering each scenario before reading the suggested workflow. Commands are examples; follow your team's branch names, merge/rebase policy, and review process. Replace example branch names with the names used in your repository.

---

## Scenario 1: Review a Colleague's PR Locally

### Situation

You are working on `feature-checkout`. A colleague has opened a pull request from `feature-payment`. You want to inspect and test their code in VS Code without merging it into your own branch.

### Your Task

Describe the safe workflow, including how you switch to the colleague's branch and return to your branch.

### Your Answer / Notes

Write your answer here:

### Suggested Workflow / Answer

1. Check Git status and save or commit your own uncommitted work.
2. Fetch the latest remote branch information.
3. Switch to the colleague's branch.
4. Review and test their code.
5. Switch back to your own branch.

If you find a bug, leave a clear PR review comment explaining the observed problem, expected behaviour, and, if useful, a test suggestion.

**Check your working tree:**

```bash
git status
```

**Fetch remote branch information:**

```bash
git fetch origin
```

**If the local branch does not exist:**

```bash
git switch --track origin/feature-payment
```

**If the local branch already exists:**

```bash
git switch feature-payment
git pull
```

**Return to your branch:**

```bash
git switch feature-checkout
```

### Important Notes

- `git fetch` does not merge changes into your current branch.
- Switching branches does not merge the colleague's code into your branch.
- Do not push changes to a colleague's branch unless you have agreed to do so.

---

## Scenario 2: Your Feature Branch Conflicts With Staging

### Situation

You created `feature-search` last week. Since then, other developers have merged changes into `staging`. You need to integrate the latest staging changes, resolve any conflicts, test your feature, and then raise a PR.

### Your Task

What workflow would you follow?

### Your Answer / Notes

Write your answer here:

### Suggested Workflow / Answer

1. Protect uncommitted work.
2. Fetch the latest remote information.
3. Switch to `feature-search`.
4. Merge `origin/staging` into your feature branch.
5. Resolve conflicts in the affected files.
6. Stage the resolved files.
7. Complete the merge if Git requires a commit.
8. Run tests and inspect the final diff.
9. Push the feature branch and open a PR targeting `staging`.

**Check status:**

```bash
git status
```

**Fetch remote updates:**

```bash
git fetch origin
```

**Switch to your feature branch:**

```bash
git switch feature-search
```

**Merge the latest remote staging into your feature branch:**

```bash
git merge origin/staging
```

**After resolving conflict markers in files:**

```bash
git add <resolved-file>
```

**Complete the merge if needed:**

```bash
git commit
```

**Test, then push:**

```bash
git push origin feature-search
```

### Important Notes

- The direction matters: run `git merge origin/staging` while on `feature-search` to bring staging into your feature branch.
- A merge does not always produce conflicts; Git may merge changes automatically.
- If your team uses rebase instead of merge, follow its policy and use the corresponding rebase conflict workflow.

---

## Scenario 3: Staging Changes After Your PR Is Opened

### Situation

Your `feature-search` branch is pushed and your PR is open. Before it is approved, another developer merges a change into `staging`.

### Your Task

Do you always need to merge the latest staging into your feature branch again? Explain when you would and would not.

### Your Answer / Notes

Write your answer here:

### Suggested Workflow / Answer

No, you do not automatically need to merge staging every time.

Update your feature branch when:

- The new staging changes affect your feature.
- The changes create compatibility or integration risks.
- Your team requires the branch to be updated.
- Required CI checks indicate that an update is necessary.

For example, if staging changes the product schema from `product_name` to `name` and your search code still uses `product_name`, update your branch, adapt your code, and test again.

If the changes are unrelated and your PR remains compatible, an update may not be necessary.

**Fetch the current remote state:**

```bash
git fetch origin
```

**Switch to your feature branch:**

```bash
git switch feature-search
```

**Merge staging if needed:**

```bash
git merge origin/staging
```

**Test and push if updated:**

```bash
git push origin feature-search
```

### Important Notes

- An existing PR normally updates automatically when you push more commits to the same source branch.
- Check repository policies and required CI checks before deciding to leave the branch unchanged.

---

## Scenario 4: You Have Uncommitted Work but Must Switch Branches

### Situation

You are on `feature-search` and have modified three files, but you have not committed them. Your manager asks you to switch to `staging` to investigate a bug.

### Your Task

What should you do before switching branches?

### Your Answer / Notes

Write your answer here:

### Suggested Workflow / Answer

Protect your changes first.

- If the work is unfinished, use `git stash`.
- If the work is at a suitable checkpoint, commit it.

Git sometimes permits switching with uncommitted changes, but it can refuse if those changes would be overwritten. Do not rely on this behaviour.

### Option A — Stash Unfinished Work

**Check status:**

```bash
git status
```

**Save your unfinished work:**

```bash
git stash push -m "WIP: feature search"
```

**Switch to staging:**

```bash
git switch staging
```

**Return later and reapply the stash:**

```bash
git switch feature-search
git stash pop
```

### Option B — Commit a Checkpoint

**Stage the relevant files:**

```bash
git add <files>
```

**Commit the checkpoint:**

```bash
git commit -m "WIP: implement product search"
```

**Switch branches:**

```bash
git switch staging
```

### Important Notes

- `git stash pop` reapplies changes and removes the stash entry if it applies successfully.
- If conflicts occur, resolve them carefully.
- Avoid blindly using `git add .` if you do not want to stage every changed file.

---

## Scenario 5: Review a Colleague's Branch Without Mixing It With Yours

### Situation

You are on `feature-search`. Your colleague has pushed commits to `origin/feature-payment`. You want to review and test their code locally without merging it into your branch.

### Your Task

Describe the workflow.

### Your Answer / Notes

Write your answer here:

### Suggested Workflow / Answer

1. Save or stash your own unfinished work.
2. Fetch remote branch information.
3. Switch to the colleague's branch.
4. Review and test their code.
5. Switch back to your branch.
6. Restore your stash if you created one.

**Protect your own changes if needed:**

```bash
git stash push -m "WIP: feature search"
```

**Fetch remote information:**

```bash
git fetch origin
```

**Create a local tracking branch if needed:**

```bash
git switch --track origin/feature-payment
```

**Or switch to an existing local branch:**

```bash
git switch feature-payment
git pull
```

**Return to your branch:**

```bash
git switch feature-search
git stash pop
```

### Important Notes

- A tracking branch links the local branch to its remote counterpart. It does not mean Git automatically reviews or merges every change.
- Only run `git stash pop` if you created a stash that you want to restore.

---

## Scenario 6: Push Rejected Because Your Branch Is Behind

### Situation

You are on `feature-search` and run:

```bash
git push origin feature-search
```

Git rejects the push because the remote branch has commits that your local branch does not have.

### Your Task

What should you do next, and why should you avoid immediately force-pushing?

### Your Answer / Notes

Write your answer here:

### Suggested Workflow / Answer

1. Check the working tree.
2. Fetch the remote changes.
3. Inspect the branch state.
4. Integrate the remote commits safely.
5. Resolve conflicts if necessary.
6. Test the result.
7. Push again.

Do not immediately force-push because it can overwrite commits pushed by another developer.

**Check the working tree:**

```bash
git status
```

**Fetch remote information:**

```bash
git fetch origin
```

**Merge the remote feature branch into your local branch:**

```bash
git merge origin/feature-search
```

**Resolve conflicts and stage resolved files:**

```bash
git add <resolved-file>
```

**Complete the merge if needed:**

```bash
git commit
```

**Run tests and push:**

```bash
git push origin feature-search
```

### Important Notes

- If your team uses rebase, the integration steps differ.
- If new commits arrive on the remote while you are working, you may need to fetch and integrate again.
- Do not use force-push as a shortcut for a non-fast-forward rejection.

---

## Scenario 7: Merge vs. Rebase

### Situation

Your manager asks you to integrate the latest staging changes into `feature-search`. One developer suggests `git merge origin/staging`; another suggests `git rebase origin/staging`.

### Your Task

Explain the main difference and when each approach is appropriate.

### Your Answer / Notes

Write your answer here:

### Suggested Workflow / Answer

**Merge**

Merge combines the histories and may create a merge commit when the branches have diverged. It preserves existing commit IDs and makes the integration point visible.

**Rebase**

Rebase replays your feature commits on top of the new base. It creates new commit IDs for the replayed commits and usually produces a more linear history.

**When to use each:**

- Merge is generally the safer default for shared branches.
- Rebase is commonly used for private or unpublished feature branches when the team prefers a linear history.
- Always follow your team's policy.

### Option A — Merge Staging Into Feature-Search

Make sure you are on `feature-search`.

```bash
git fetch origin
git switch feature-search
git merge origin/staging
```

### Option B — Rebase Feature-Search Onto Staging

Use this only when appropriate for your workflow.

```bash
git fetch origin
git switch feature-search
git rebase origin/staging
```

### Important Notes

- Do not casually rebase commits that other developers are already using. Rebase rewrites history and can disrupt collaborators.
- During a merge conflict, resolve the files, stage them with `git add`, and complete the merge if required.
- During a rebase conflict, resolve the files, stage them, and continue:

```bash
git add <resolved-file>
git rebase --continue
```

- To cancel an in-progress rebase:

```bash
git rebase --abort
```

- After rebasing a branch that has already been pushed, coordinate with your team before pushing rewritten history. Do not force-push blindly.

---

# Quick Git Command Reference

| Purpose | Command |
|---|---|
| Check working tree | `git status` |
| Fetch remote updates | `git fetch origin` |
| Switch branch | `git switch <branch-name>` |
| Create and track a remote branch | `git switch --track origin/<branch-name>` |
| Merge staging into the current feature branch | `git merge origin/staging` |
| Stash unfinished changes | `git stash push -m "WIP: description"` |
| Reapply the latest stash | `git stash pop` |
| Stage a resolved file | `git add <file>` |
| Push a feature branch | `git push origin <branch-name>` |
| Continue after resolving a rebase conflict | `git rebase --continue` |
| Abort an in-progress rebase | `git rebase --abort` |

---

# Core Rules to Remember

1. **Fetch is not merge.** `git fetch` updates remote-tracking information; it does not merge those changes into your current branch.
2. **Your current branch receives the integration.** The branch you are currently on is the branch that receives the merge or rebase result.
3. **Check the direction.** To bring staging into `feature-search`, switch to `feature-search` first, then merge or rebase `origin/staging`.
4. **Protect uncommitted work.** Before switching branches, check `git status` and protect your changes.
5. **Test after integration.** Resolve conflicts, test the result, inspect the diff, then push and raise or update the PR.
6. **Follow team policy.** Use merge or rebase according to your team's workflow. Avoid rewriting shared history without coordination.
7. **Write useful PR reviews.** Explain the issue, expected behaviour, and ideally how to reproduce or test it.
