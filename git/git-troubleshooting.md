# Git Repository Corruption Recovery

## Issue

While committing my Docker Compose learning log, Git started reporting the following errors:

```text
error: object file .git/objects/... is empty
fatal: bad object HEAD
```

Git commands such as:

```bash
git status
git commit
```

were no longer working because the local Git repository had become corrupted.

---

## Root Cause

The exact cause could not be determined, but the repository contained corrupted Git object files and an invalid `HEAD` reference.

Possible causes include:

- Interrupted Git operation
- Unexpected WSL shutdown
- Filesystem corruption
- Disk write interruption

---

## Investigation

Checked repository integrity:

```bash
git fsck --full
```

Verified Git reported:

- Invalid HEAD
- Empty object files
- Corrupted object database

---

## Resolution

Since the GitHub repository was healthy:

1. Renamed the corrupted local repository.

```bash
mv devops-foundations devops-foundations-broken
```

2. Cloned a fresh copy.

```bash
git clone git@github.com:RaghavN6666/devops-foundations.git
```

3. Compared the backup with the fresh clone.

4. Restored any uncommitted files if necessary.

5. Verified repository health.

```bash
git status
```

Output:

```text
On branch main
Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
```

---

## Lessons Learned

- Push work regularly to GitHub.
- GitHub serves as a reliable backup for repository recovery.
- Preserve the corrupted repository until recovery is verified.
- Use `git fsck` to diagnose repository corruption.
- A fresh clone is often the fastest recovery method when the remote repository is healthy.