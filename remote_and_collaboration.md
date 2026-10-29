# Git Advanced Learning — Remotes & Collaboration

## Part 1: Concepts

### 1. `git fetch` vs `git pull` vs `git pull --rebase`

These commands retrieve updates from a remote repository, but they differ in how they integrate those updates.

| Command | What it does |
|---|---|
| `git fetch origin` | Downloads remote updates and updates remote-tracking references without integrating them into the current branch. |
| `git pull` | Fetches updates and integrates them using the configured/default integration behavior. |
| `git pull --rebase` | Fetches updates and rebases local commits on top of the fetched branch. |

#### Example

Your colleague pushes two commits to GitHub. You want to inspect them before changing your local `main`.

```bash
git fetch origin
git log --oneline main..origin/main
```

The log command shows commits reachable from `origin/main` but not from your local `main`.

To compare changes:

```bash
git diff main..origin/main
```

After reviewing the updates, decide how to integrate them.

**Remember:** Fetch first to inspect. Pull to fetch and integrate.

### 2. Configuring `pull.rebase`

To make rebasing the default pull behavior for your user account:

```bash
git config --global pull.rebase true
```

To apply it only to the current repository:

```bash
git config --local pull.rebase true
```

To prefer merge-based pulls globally:

```bash
git config --global pull.rebase false
```

To check the effective value:

```bash
git config --get pull.rebase
```

`--global` affects your user-level Git configuration across repositories, unless a more specific setting overrides it. `--local` affects only the current repository.

A global setting is a preference, not a team-wide configuration. Choose it according to your team's workflow.

### 3. Upstream tracking

An upstream tracking branch is the remote branch that a local branch tracks by default.

For example, local `feature-search` may track `origin/feature-search`.

Check your tracking configuration:

```bash
git branch -vv
```

Set an upstream when pushing a new branch for the first time:

```bash
git push -u origin feature-search
```

The `-u` option sets the upstream tracking relationship. Afterward, Git often knows which remote branch to use when you run `git pull` or `git push` without specifying the remote and branch.

**Important:** An upstream tracking branch is not the same concept as a remote named `upstream`.

- **Upstream tracking branch:** the remote branch a local branch tracks.
- **Remote named `upstream`:** commonly the original repository in a fork workflow.

### 4. Multiple remotes: `origin` and `upstream`

A remote is a named connection to another repository.

List configured remotes:

```bash
git remote -v
```

In a common fork-based workflow:

- `origin` points to your fork.
- `upstream` points to the original repository.

Fetch from the original repository:

```bash
git fetch upstream
```

This downloads updates but does not automatically merge them into your local branch.

You can then inspect the changes and decide whether to merge or rebase them.

These names are conventions, not mandatory Git keywords. In a shared-repository workflow, `origin` commonly points directly to the team's repository, and there may be no `upstream` remote.

### 5. Fork-and-PR model vs shared-repository model

#### Fork-and-PR model

1. Fork the original repository on GitHub.
2. Clone your fork.
3. Create a feature branch.
4. Commit and push changes to your fork.
5. Open a pull request against the original repository.
6. Review and merge the pull request.

This model is common in open-source projects and repositories where contributors do not have direct write access.

#### Shared-repository model

1. Clone the team's repository.
2. Create a feature branch.
3. Commit and push your branch to the shared repository.
4. Open a pull request targeting the team's main branch.
5. Review and merge the pull request.

This model is common when team members have write access to the same repository.

**Important:** Both models can use feature branches and pull requests. The main difference is whether you push to your own fork or to a branch in the shared repository.

### 6. Deleting remote branches and pruning stale references

Delete a remote branch:

```bash
git push origin --delete feature-search
```

This asks the remote named `origin` to delete the branch `feature-search`.

After a remote branch is deleted, your local repository may still display a stale remote-tracking reference, such as `origin/feature-search`.

Fetch updates and remove stale references:

```bash
git fetch --prune
```

This updates remote-tracking information and removes stale references for branches that no longer exist on the remote.

Alternatively, prune stale references for a specific remote without fetching new updates:

```bash
git remote prune origin
```

**Important:** Pruning stale remote-tracking references does not normally delete your local branch. A local branch named `feature-search` is separate from the remote-tracking reference `origin/feature-search`.

### 7. SSH keys vs HTTPS with credential helpers

GitHub supports both SSH and HTTPS for Git operations.

#### SSH

SSH authentication uses a key pair:

- A private key stays on your computer.
- A public key is added to your GitHub account or other authorized location.
- Git uses the key to authenticate.

Check your remote URL:

```bash
git remote -v
```

An SSH URL commonly looks like:

```text
git@github.com:OWNER/REPOSITORY.git
```

#### HTTPS

An HTTPS URL commonly looks like:

```text
https://github.com/OWNER/REPOSITORY.git
```

GitHub generally requires a personal access token or another supported authentication method instead of an account password for HTTPS Git operations.

A credential helper can securely store or retrieve credentials so you don't have to enter them repeatedly. Git Credential Manager is one example.

### 8. Fine-grained Personal Access Tokens (PATs)

A fine-grained PAT can limit access to selected repositories and permissions.

Good practices:

- Grant only the permissions required.
- Select only the repositories needed.
- Set an appropriate expiration.
- Store credentials using a trusted credential helper.
- Never commit tokens to source code or configuration files.
- Revoke an exposed token and create a replacement.

SSH keys are another valid authentication method; PATs are not required for ordinary Git operations when SSH authentication is used.

---

## Part 2: Practical Scenarios and Situations

### Scenario 1: Inspect remote updates without integrating them

**Situation:** Your colleague pushed new commits to `main`. You want to download the updated remote information but keep your current local branch unchanged.

**Question:** What should you run?

**Answer:**

```bash
git fetch origin
```

Then inspect incoming commits:

```bash
git log --oneline main..origin/main
```

**Why:** Fetch updates your knowledge of the remote without integrating the commits into local `main`.

**Why not `git pull --rebase`?** It downloads and integrates updates by rebasing, which is more than you requested.

### Scenario 2: Rebase local commits when pulling

**Situation:** Your team prefers linear history. Your local branch and the remote branch both have new commits, and you want to rebase your local commits on top of the fetched remote branch.

**Question:** What command should you use for this pull?

**Answer:**

```bash
git pull --rebase
```

**Why:** It fetches remote updates and replays your local commits on top of the updated branch.

**Caution:** If conflicts occur, resolve them carefully. Use `git rebase --abort` if you need to cancel an in-progress rebase.

### Scenario 3: Set a global pull preference

**Situation:** You want Git to prefer rebase-based pulls by default across your repositories.

**Answer:**

```bash
git config --global pull.rebase true
```

**Why not `--local`?** That applies only to the current repository.

**Why not `git pull --rebase`?** That affects a single pull, rather than saving a global preference.

### Scenario 4: Push a new feature branch and set tracking

**Situation:** You created `feature-search` locally and want to push it to GitHub while configuring its tracking branch.

**Answer:**

```bash
git push -u origin feature-search
```

**Why:** The branch is pushed to `origin`, and `-u` sets the upstream tracking relationship.

**Why not just `git fetch`?** Fetch downloads remote information; it doesn't publish your local branch.

### Scenario 5: Update your fork from the original repository

**Situation:** You contribute through a fork. The original project has new commits, and you want to retrieve them.

**Answer:**

```bash
git fetch upstream
```

**Why:** In this workflow, `upstream` commonly points to the original repository.

**Why not assume `origin` means the original repository?** In a fork workflow, `origin` commonly points to your own fork. Always verify with `git remote -v`.

### Scenario 6: Choose between a fork and a shared repository

**Situation A:** You contribute to an open-source repository and don't have permission to push branches to the original repository.

**Approach:** Fork the repository, push your feature branch to your fork, and open a pull request against the original project.

**Situation B:** You work with teammates who have write access to the same repository.

**Approach:** Create a feature branch, push it to the shared repository, and open a pull request for review.

**Key difference:** The destination for your feature branch is your fork in the first workflow and the shared repository in the second.

### Scenario 7: Delete a remote branch and clean up

**Situation:** Your pull request has been merged, and you want to delete the remote feature branch.

**Answer:**

```bash
git push origin --delete feature-search
git fetch --prune
```

**Why:** The first command deletes the branch from the remote. The second updates your remote-tracking references and removes stale ones.

**Why not `git branch -d feature-search`?** That deletes a local branch, not the remote branch.

### Scenario 8: A deleted remote branch still appears locally

**Situation:** Your colleague deleted `feature-cart` from GitHub, but `origin/feature-cart` still appears locally.

**Answer:**

```bash
git fetch --prune
```

**Why:** Your local repository had a stale remote-tracking reference that needed to be cleaned up.

**Why not `git reset --hard`?** Reset changes the current branch and working tree; it isn't the command for pruning stale remote-tracking references.

### Scenario 9: GitHub authentication over HTTPS

**Situation:** Git asks for credentials when you push over HTTPS.

**Approach:**

- Use a supported authentication method, such as a fine-grained PAT.
- Grant only the required repository permissions.
- Use a trusted credential helper to store or retrieve credentials.
- Never place the token in your source code or commit it to Git.

**Alternative:** Configure SSH keys and use an SSH remote URL.

### Scenario 10: Choose a safer push after a rebase

**Situation:** You rebased your feature branch and need to push the rewritten history. The remote may have changed since your last fetch.

**Approach:**

1. Fetch the latest remote information.
2. Inspect the remote branch and determine whether a teammate added commits.
3. Coordinate with teammates if the branch is shared.
4. Only if rewriting the remote history is appropriate, consider `git push --force-with-lease`.

**Why not `git push --force`?** It does not provide the same expected-remote-state safety check and can overwrite remote work.

**Important:** `--force-with-lease` is safer, not risk-free. Inspect the current remote state before using it.

---

## Part 3: Quick Revision Questions

1. What is the difference between `fetch` and `pull`?
2. What does `git pull --rebase` do?
3. What is the difference between `--local` and `--global` Git configuration?
4. What does `git push -u origin feature-search` configure?
5. In a fork workflow, what do `origin` and `upstream` commonly represent?
6. What is the difference between a remote branch and a remote-tracking reference?
7. Which command deletes a branch from a remote?
8. What does `git fetch --prune` clean up?
9. What is the difference between SSH authentication and HTTPS authentication?
10. Why is `git push --force-with-lease` safer than `git push --force`?

## Key Takeaway

- Fetch first when you want to inspect remote updates.
- Pull when you are ready to integrate remote updates.
- Use upstream tracking to make routine push and pull operations easier.
- Understand whether you are working through a fork or a shared repository.
- Delete remote branches deliberately and prune stale references when needed.
- Protect credentials with SSH keys or carefully scoped tokens and a trusted credential helper.
- Use `--force-with-lease` cautiously when rewriting remote history.
