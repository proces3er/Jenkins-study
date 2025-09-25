Sure! Here's your content cleaned up and formatted nicely as a Markdown (`.md`) file:

````markdown
# Git: Upload Local File to Remote Branch `test1`

This guide explains how to upload a local file to a remote GitHub branch (`test1`) using Git CLI.

---

## 1. Check your remote setup

Run:

```bash
git remote -v
````

This shows your configured remotes.

---




## 2. Add a remote repository (if not already set)

```bash
git remote add origin https://github.com/proces3er/Jenkins-study.git
```

Verify:

```bash
git remote -v
```

You should see:

```
origin  https://github.com/proces3er/Jenkins-study.git (fetch)
origin  https://github.com/proces3er/Jenkins-study.git (push)
```


 if local file and remote branch exist
---

## 3. Upload your file to the `test1` branch

### Step 3.1: Fetch the remote branches

```bash
git fetch origin
```

### Step 3.2: Check out the `test1` branch from remote

```bash
git checkout -b test1 origin/test1
```

> If you just want to create a new branch called `test1` locally and push it later, you can skip the `origin/test1` part. The files from the remote branch are downloaded to your local working directory, and your branch is ready for edits.

### Step 3.3: Stage and commit your file

```bash
git add test.txt
git commit -m "Add test.txt"
```

### Step 3.4: Push to the remote `test1` branch

```bash
git push origin test1
```




## If created a local branch that does not have a common ancestor commit
---
if local file and remote branch (test1) doesnt exist
## 4. Set upstream tracking (optional but recommended)

If your branch has no upstream, set it:

```bash
git push --set-upstream origin test1
# or shorthand
git push -u origin test1
```

* `-u` / `--set-upstream` tells Git:
  *“From now on, local `test1` will track `origin/test1`.”*

After this, you can simply use:

```bash
git push
git pull
```

without specifying the remote branch.

---

**Now your local file is uploaded to the remote branch `test1` and your local branch is tracking it.**

```

---










### Handling Non-Fast-Forward Pushes

Sometimes, when you try to push a local branch to a remote branch, Git may reject the push with a **non-fast-forward** error:

```

! \[rejected] feature2 -> feature (non-fast-forward)
Updates were rejected because a pushed branch tip is behind its remote counterpart.

````

This happens because the **remote branch already contains commits** that your local branch does not have. Git prevents the push to avoid overwriting history on the remote.

#### How to Fix

````
There are two main ways to integrate the remote changes:

**Option 1: Merge Remote into Local Branch**

```bash
git fetch origin
git checkout feature2
git merge origin/feature   # Merge remote changes into your local branch
# Resolve any conflicts if necessary
git push origin feature2:feature

````
This keeps the full history and creates a merge commit if there are new commits on the remote.

**Option 2: Rebase Local Branch on Top of Remote (Linear History)**

```bash
git fetch origin
git checkout feature2
git rebase origin/feature
# Resolve conflicts if any:
git add <file>
git rebase --continue
git push origin feature2:feature
```

Rebasing replays your local commits **on top of the remote branch**, producing a cleaner, linear commit history.

#### Important Note

Avoid using `git push --force` unless you are certain you want to overwrite the remote branch, as this can erase commits made by others.

```

---

