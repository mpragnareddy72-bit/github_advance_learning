# 1. Git Internals

## 1.1 Git Workflow: Working Tree, Staging Area, Local Repository, and Remote Repository

### Step 1: Working Tree
When you modify the code in `rag.py`, the changes exist in your working tree.

### Step 2: Staging Area
When you run:

```bash
git add rag.py
```

Git moves the selected changes to the staging area, preparing them for a commit.

### Step 3: Local Repository
When you run:

```bash
git commit -m "Add RAG retrieval"
```

Git saves a snapshot of the staged changes in your local repository.

### Step 4: Remote Repository
When you run:

```bash
git push
```

Git sends your local commits to the remote repository, such as GitHub, provided the remote and upstream branch are configured correctly.

## 1.2 Git Objects

Git uses four main types of objects to store and organize repository data.

| Object | What Does It Do? | Example |
|---|---|---|
| Blob | Stores the content of a file. | The content of `rag.py` |
| Tree | Represents the structure of files and directories. | The project folder structure |
| Commit | Stores a reference to a project snapshot's tree, its parent commit(s), author information, commit message, and other metadata. | `Add RAG retrieval` |
| Annotated Tag | Gives a specific commit a named label and stores additional metadata, such as the tagger, date, and message. | `v1.0.0` |

## 1.3 HEAD, Detached HEAD, ORIG_HEAD, FETCH_HEAD, and Tags

### 1. HEAD

`HEAD` tells Git which branch or commit you are currently working from.

**Example:**

```bash
git switch feature/login
```

Now, you are working on the `feature/login` branch. Normally, `HEAD` points to that branch, which in turn points to its latest commit.

### 2. Detached HEAD

A Detached HEAD state occurs when you check out a specific commit directly instead of working through a branch.

**Example:**

```bash
git checkout a1b2c3d
```

Now, you can inspect the project at that particular commit.

However, if you create a new commit in this state, it will not automatically belong to an existing branch. If no branch or other reference points to that commit, it may eventually become difficult to find.

**Important:** If you want to keep making changes, create a new branch from the detached commit:

```bash
git switch -c feature/recover-work
```

This creates a branch at the current commit and switches to it.

### 3. ORIG_HEAD

`ORIG_HEAD` stores the previous position of `HEAD` before certain major Git operations.

For example, during operations such as `git reset`, Git may save the previous commit in `ORIG_HEAD`.

This can help you return to an earlier commit if you need to recover from an unintended operation.

However, `ORIG_HEAD` is not a permanent backup. Other operations may overwrite it.

### 4. FETCH_HEAD

`FETCH_HEAD` stores information about the commits and references fetched during the most recent fetch operation.

**Example:**

```bash
git fetch origin
```

Git retrieves information about changes from the remote repository named `origin`.

`FETCH_HEAD` records information about what was fetched. It does not automatically merge those changes into your current branch.

### 5. Tags

A tag gives a name to an important commit.

**Example:**

```bash
git tag v1.0
```

You can use this tag to identify an important version of your FastAPI project.

For example, `v1.0` might represent the first stable release of your project.

To view your tags:

```bash
git tag
```

**Note:** The command above creates a lightweight tag. An annotated tag can be created using:

```bash
git tag -a v1.0.0 -m "First stable release"
```

An annotated tag stores additional metadata, including the tag message and tagger information.

## 1.4 .git/, Packfiles, git gc, git fsck, and Reflog

### 1. `.git/`

The `.git/` directory contains Git's internal metadata and repository information.

It includes information about commits, branches, references, configuration, and object storage.

**Important:** Do not directly edit or delete files inside `.git/` unless you understand exactly what you are doing.

### 2. Packfiles

Packfiles store Git objects efficiently by compressing them and, where appropriate, storing objects as differences from other objects.

This helps reduce the amount of disk space required by a repository.

### 3. `git gc`

The `git gc` command performs Git repository maintenance.

It can optimize storage by packing objects and performing cleanup when appropriate.

You do not need to run `git gc` manually during your normal daily development workflow.

### 4. `git fsck`

The `git fsck` command checks the integrity and connectivity of objects in the Git object database.

It can help identify missing, corrupted, or unreachable objects.

### 5. Reflog

The reflog records changes to local Git references, including movements of `HEAD`.

It can help you recover commits or previous branch positions after certain accidental operations.

**Useful practice commands:**

```bash
git rev-parse --show-toplevel
git log --oneline -5
git cat-file -t HEAD
git reflog -5
```

**What these commands do:**

- `git rev-parse --show-toplevel` — Displays the root directory of the current Git repository.
- `git log --oneline -5` — Displays the five most recent commits in a compact format.
- `git cat-file -t HEAD` — Displays the type of Git object referenced by `HEAD`. It usually prints `commit`.
- `git reflog -5` — Displays the five most recent reflog entries.

**Important safety notes:**

- You do not need to run `git gc` manually right now.
- Do not directly edit or delete files inside `.git/`.
- Use `git reflog`