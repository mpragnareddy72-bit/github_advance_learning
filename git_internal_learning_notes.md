# Git Internals — Complete Learning Notes

## Table of Contents

1. Git's Three Areas
2. Git Object Model and Hashing
3. Commits: Snapshots, Parent Pointers, and Diffs
4. Git References (Refs)
5. Git Objects, Packfiles, Garbage Collection, and Integrity Checks
6. Git Reflog and Recovery

---

## 1. Git's Three Areas

Git has three important areas:

### 1.1 Working Tree

The working tree is the project directory where we create, edit, and delete files.

Example: modifying `main.dart` in VS Code.

Git detects changes to tracked files, but those changes are not automatically staged or committed.

### 1.2 Staging Area (Index)

The staging area contains the changes selected for the next commit.

Command:

```bash
git add main.dart
```

This stages changes to `main.dart`.

To stage all changes in the current directory:

```bash
git add .
```

`git add` does not create a commit. It prepares changes for the next commit.

### 1.3 Repository (`.git`)

The `.git` directory stores Git's object database, references, configuration, and other repository information.

To record staged changes in the repository:

```bash
git commit -m "Update main screen"
```

### Workflow

```text
Working Tree
     |
     | git add
     v
Staging Area
     |
     | git commit
     v
Local Repository
     |
     | git push
     v
Remote Repository (GitHub)
```

Useful command:

```bash
git status
```

It shows the current branch and the state of working-tree and staged changes.

---

## 2. Git Object Model and Hashing

Git stores project data using four important object types.

### 2.1 Blob

A blob stores the contents of a file.

It does not store the original filename or directory location.

If two files have identical contents, Git can reuse the same blob object.

### 2.2 Tree

A tree represents a directory's contents.

It stores entries containing filenames, file modes, object types, and references to blobs or other trees.

A project containing multiple directories is represented using connected tree objects.

### 2.3 Commit

A commit records a project snapshot and metadata, including:

- A reference to the top-level tree.
- Author and committer information.
- Commit message.
- Parent commit reference(s), except for a root commit.

### 2.4 Annotated Tag

An annotated tag is a separate Git object containing:

- Tag name.
- Tagger information.
- Date.
- Message.
- Reference to the tagged object.

Annotated tags are commonly used to mark software releases.

Create an annotated tag:

```bash
git tag -a v1.0.0 -m "First release"
```

List tags:

```bash
git tag
```

A lightweight tag is different: it is simply a reference to an object and does not create an annotated tag object.

### 2.5 Hashing: SHA-1 vs SHA-256

Git calculates a hash for an object using its type, size, and content.

The hash acts as an object identifier.

| Feature | SHA-1 | SHA-256 |
|---|---|---|
| Hash length | 160 bits | 256 bits |
| Hexadecimal characters | 40 | 64 |
| Collision resistance | Known weaknesses | Stronger |
| Git support | Supported | Supported |

SHA-1 has known collision weaknesses. SHA-256 offers stronger collision resistance.

Check your repository's object format:

```bash
git rev-parse --show-object-format
```

The result is commonly `sha1` for traditional Git repositories.

Inspect the latest commit:

```bash
git log --oneline -5
```

Git displays abbreviated commit hashes for readability.

---

## 3. A Commit Is a Snapshot Plus Parent Pointers, Not a Diff

A commit represents a snapshot of the tracked project files at a particular point in history.

It also records its parent commit reference(s) and metadata.

Example:

```text
Commit C: Update main screen
        |
        | parent
        v
Commit B: Add settings screen
        |
        | parent
        v
Commit A: Initial application
```

Each newer commit points backward to its parent.

A root commit has no parent. A merge commit can have multiple parents.

### What is a snapshot?

Suppose Commit A contains:

```dart
print('Hello');
```

Commit B contains:

```dart
print('Hello Buddy');
```

Commit B records the newer snapshot, while its parent reference identifies Commit A.

Git does not define a commit as merely the changes between these two versions.

Git can calculate the diff by comparing their snapshots.

View the latest commit:

```bash
git show --no-patch HEAD
```

Compare the latest two commits:

```bash
git diff HEAD~1 HEAD
```

Inspect the commit's tree and parent information:

```bash
git show --no-patch --pretty=raw HEAD
```

A commit contains a reference to its top-level tree, parent reference(s), and metadata. Git can reuse existing objects rather than storing a separate physical copy of every unchanged file for every commit.

---

## 4. Git References (Refs)

References are names or pointers that help Git locate objects, especially commits, without requiring full hashes.

### 4.1 Branches

A branch is a movable reference that normally points to the latest commit on a line of development.

Examples:

```text
main
staging
settings_fontsize
```

List local branches:

```bash
git branch
```

Show the current branch:

```bash
git branch --show-current
```

Switch branches:

```bash
git switch staging
```

Create a branch at the current commit:

```bash
git branch experiment
```

### 4.2 Tags

Tags identify particular points in history, commonly software releases.

Unlike branches, tags are generally intended to remain fixed.

```bash
git tag -a v1.0.0 -m "First release"
git tag
```

### 4.3 HEAD

`HEAD` identifies the current checkout.

In a normal checkout, it usually points symbolically to the current branch.

Example:

```text
HEAD -> main -> Commit C
```

Inspect the current branch reference:

```bash
git symbolic-ref HEAD
```

View the latest commit:

```bash
git log -1 --oneline
```

### 4.4 Detached HEAD

In a detached HEAD state, `HEAD` points directly to a commit instead of a branch.

```bash
git switch --detach <commit-hash>
```

This is useful for inspecting or testing an older commit.

If you create commits in this state, they are not automatically attached to a branch. To preserve new work, create a branch:

```bash
git switch -c experiment
```

Replace `<commit-hash>` with an actual commit hash.

### 4.5 ORIG_HEAD

`ORIG_HEAD` is a special reference Git may save before certain operations that move `HEAD`, such as resets, merges, or rebases.

Inspect it:

```bash
git show --no-patch --oneline ORIG_HEAD
```

It can help locate a previous commit after an operation goes wrong.

It is not a permanent backup and may be overwritten by later operations.

### 4.6 FETCH_HEAD

`FETCH_HEAD` records information about commits or references fetched from a remote repository.

Fetch updates from GitHub:

```bash
git fetch origin
```

Inspect the fetched commit information:

```bash
git show --no-patch --oneline FETCH_HEAD
```

`FETCH_HEAD` is different from `origin/main`.

- `origin/main` is a remote-tracking branch.
- `FETCH_HEAD` records information about a particular fetch and can be overwritten by another fetch.

---

## 5. Git Objects, Packfiles, Garbage Collection, and Integrity Checks

### 5.1 `.git/objects`

Git stores its object database inside:

```text
.git/objects/
```

Objects include blobs, trees, commits, and annotated tags.

Git can store them as loose objects or in packfiles.

### 5.2 Loose Objects

A loose object is an individual compressed Git object.

In a SHA-1 repository, Git uses the first two characters of the object's hash as a directory name and the remaining 38 characters as its filename.

Example:

```text
.git/objects/1a/2b3c4d...
```

The contents are compressed object data, not necessarily readable source code.

### 5.3 Packfiles

Packfiles combine multiple Git objects into a more efficient storage format.

Location:

```text
.git/objects/pack/
```

Common files:

```text
pack-xxxx.pack
pack-xxxx.idx
```

- `.pack` stores packed objects.
- `.idx` helps Git locate objects in the pack.

Packfiles can use compression and delta compression to reduce storage requirements.

### 5.4 Git Garbage Collection

`git gc` performs repository maintenance.

It may:

- Pack loose objects.
- Repack objects more efficiently.
- Remove eligible temporary or unreachable objects.
- Perform other maintenance tasks.

Command:

```bash
git gc
```

Git can also perform maintenance automatically.

Garbage collection is not a backup system. Unreachable objects can eventually be pruned.

### 5.5 Git File System Check

`git fsck` checks the integrity and connectivity of Git objects.

Command:

```bash
git fsck
```

A fuller check:

```bash
git fsck --full
```

It can detect missing or corrupt objects and report unreachable or dangling objects.

Dangling objects are not necessarily corrupt; they may be valid objects no longer referenced by the current history.

### Useful Diagnostic Commands

```bash
git count-objects -vH
git cat-file -t HEAD
git cat-file -p HEAD
```

- `git count-objects -vH` displays object counts and storage information.
- `git cat-file -t HEAD` shows the object type referenced by HEAD.
- `git cat-file -p HEAD` displays the object's readable contents.

---

## 6. Git Reflog: A Safety Net for Recovery

The reflog records movements of references such as `HEAD` and local branches.

It can help recover a commit after an accidental reset, rebase, or branch movement.

### 6.1 View the Reflog

```bash
git reflog
```

Example:

```text
abc1234 HEAD@{0}: reset: moving to HEAD~2
def5678 HEAD@{1}: commit: Update settings screen
ghi9012 HEAD@{2}: commit: Add voice selection
```

The hashes and messages above are illustrative.

- `HEAD@{0}` is the most recent entry.
- `HEAD@{1}` is the previous entry.
- `HEAD@{2}` is an older entry.

View relative timestamps:

```bash
git reflog --date=relative
```

View the reflog for a particular branch:

```bash
git reflog show settings_fontsize
```

### 6.2 Recover a Commit

If an accidental reset moves your branch backward, the reflog may still contain the previous commit.

First, inspect the reflog:

```bash
git reflog
