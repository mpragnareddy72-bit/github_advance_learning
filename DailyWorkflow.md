# Git Commands and Workflow — Complete Learning Notes

## Table of Contents

1. [Git Repository Basics](#1-git-repository-basics)
2. [git init](#2-git-init)
3. [git clone](#3-git-clone)
4. [git status](#4-git-status)
5. [git add -p](#5-git-add--p)
6. [git commit --amend](#6-git-commit---amend)
7. [git restore](#7-git-restore)
8. [git switch](#8-git-switch)
9. [Atomic Commits and Conventional Commits](#9-atomic-commits-and-conventional-commits)
10. [git diff vs git diff --staged vs git diff branchA...branchB](#10-git-diff-vs-git-diff---staged-vs-git-diff-brancha...branchb)
11. [Git Stash](#11-git-stash)
12. [Git .gitignore](#12-git-gitignore)
13. [Global Git Ignore](#13-global-git-ignore)
14. [git check-ignore -v](#14-git-check-ignore--v)
15. [Complete Git Command Revision](#15-complete-git-command-revision)

---

## 1. Git Repository Basics

Git helps track changes in a project, maintain history, and collaborate with other developers.

Git has three important areas:

1. **Working tree:** The files and folders you are currently editing.
2. **Staging area (index):** The changes selected for your next commit.
3. **Repository history:** The commits saved in Git.

### Basic workflow

```text
Working Tree
     |
     | git add
     v
Staging Area
     |
     | git commit
     v
Repository History
```

Example:

```powershell
git status
git add main.py
git diff --staged
git commit -m "feat: add main functionality"
```

- `git status` checks the current state.
- `git add` stages changes.
- `git diff --staged` reviews staged changes.
- `git commit` records a snapshot.

---

## 2. git init

### What is `git init`?

`git init` initializes a new Git repository in the current directory.

Use it when you already have a project on your computer and want to start tracking its changes with Git.

### Example

```powershell
mkdir folder_name
cd folder_name
git init
```

Git creates a hidden `.git` directory containing repository metadata, objects, and references.

### Important points

- `git init` does not automatically stage files.
- It does not create a commit.
- It does not upload anything to GitHub.
- You can check the repository state using `git status`.

---

## 3. git clone

### What is `git clone`?

`git clone` copies an existing Git repository to your computer, including its history by default.

```powershell
git clone https://github.com/username/project.git
cd project
```

Replace the example URL with the actual repository URL.

### 3.1 Clone with `--depth`

```powershell
git clone --depth 1 https://github.com/username/project.git
```

`--depth 1` creates a shallow clone containing only the latest commit history initially.

**Advantages:**
- Can reduce download time.
- Can reduce the amount of history downloaded.
- Useful when only the latest version is needed.

**Limitation:** Older commit history is not initially available.

### 3.2 Clone with `--filter=blob:none`

```powershell
git clone --filter=blob:none https://github.com/username/project.git
```

This creates a partial clone that initially avoids downloading file contents stored as Git blob objects. Git can fetch the required contents later.

Remember the Git object types:

- **Blob:** File content.
- **Tree:** Directory and filename information.
- **Commit:** A snapshot with metadata and parent pointers.

This option is useful for large repositories where downloading all file contents immediately is unnecessary. The server must support the relevant filtering features.

### 3.3 Combine both options

```powershell
git clone --depth 1 --filter=blob:none https://github.com/username/project.git
```

This requests both shallow history and blob filtering.

---

## 4. git status

### What is `git status`?

```powershell
git status
```

It shows the current state of your files, including:

- Untracked files.
- Modified files that are not staged.
- Staged changes ready to commit.
- The current branch and relevant branch status.

### Example

```text
Changes not staged for commit:
    modified: main.py

Changes to be committed:
    modified: README.md

Untracked files:
    notes.txt
```

Meaning:

- `main.py` has changes that are not staged.
- `README.md` has staged changes.
- `notes.txt` is a new, untracked file.

**Best practice:** Run `git status` before staging, committing, switching branches, or restoring files.

---

## 5. git add -p

### What is `git add -p`?

The `-p` option means patch mode.

It lets you select individual sections of changes, called *hunks*, instead of staging every change in a file.

```powershell
git add -p main.py
```

### Common options

| Option | Meaning |
|---|---|
| `y` | Stage this hunk |
| `n` | Do not stage this hunk |
| `s` | Split the hunk into smaller hunks, if possible |
| `e` | Manually edit the patch |
| `q` | Quit patch mode |

### Example

Suppose `main.py` contains changes for two unrelated tasks:

- Adding a new feature.
- Fixing an unrelated bug.

You can stage only the feature changes:

```powershell
git add -p main.py
git diff --cached
git commit -m "feat: add new feature"
```

Then stage the remaining bug fix separately.

### Why use it?

- Create atomic commits.
- Avoid committing unrelated changes together.
- Review and control exactly what enters the next commit.

---

## 6. git commit --amend

### What is `git commit --amend`?

It replaces the most recent commit with a new commit containing the updated content and/or commit message.

### 6.1 Correct the previous commit message

```powershell
git commit --amend -m "feat: add voice selection screen"
```

This changes the previous commit message and creates a replacement commit.

### 6.2 Include additional changes in the previous commit

```powershell
git add another_file.py
git commit --amend --no-edit
```

`--no-edit` keeps the previous commit message while including the newly staged changes.

### Important points

- Amending changes the commit hash.
- Git creates a replacement commit rather than editing the old commit in place.
- Avoid amending commits already shared with others unless you understand the consequences of rewriting history.
- Check `git status` and `git log --oneline` before and after amending.

---

## 7. git restore

`git restore` is used to restore file contents or remove changes from the staging area.

### 7.1 Discard unstaged changes

```powershell
git restore main.py
```

This restores the working-tree version of `main.py` from the index.

If the file has no staged changes, this normally restores it to the committed version.

**Warning:** Unstaged changes to that file can be permanently lost.

### 7.2 Unstage changes

```powershell
git restore --staged main.py
```

This removes the staged changes from the index but keeps your working-tree edits.

It does not delete the edits from the file.

### Difference

| Command | Result |
|---|---|
| `git restore main.py` | Discards unstaged edits by restoring from the index |
| `git restore --staged main.py` | Unstages changes but keeps working-tree edits |

### Example workflow

```powershell
git add main.py
git status
git restore --staged main.py
git status
```

After the last command, the changes remain in the working tree but are no longer staged.

---

## 8. git switch

### What is `git switch`?

`git switch` changes the branch you are working on.

Branches let you develop different features independently.

### 8.1 List local branches

```powershell
git branch
```

The current branch is marked with an asterisk (`*`).

### 8.2 Switch to an existing branch

```powershell
git switch staging
```

This switches to the local `staging` branch.

### 8.3 Create a new branch and switch to it

```powershell
git switch -c settings_fontsize
```

The `-c` option creates a new branch and switches to it.

### Example workflow

```powershell
git switch staging
git switch -c