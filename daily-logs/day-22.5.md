# 📅 Daily Learning Log

**Date:** July 29, 2026

## 🎯 Objective
Investigate and recover from a corrupted local Git repository while preserving project history and files.

---

## 📚 Topics Covered
- Git Repository Corruption
- Git Object Database
- `git fsck`
- Repository Recovery
- Cloning from GitHub
- Local vs Remote Repository

---

## 🧠 Concepts Learned

### 1. Git Repository Corruption
- A Git repository can become corrupted if its internal object database is damaged.
- Symptoms included:
  - `fatal: bad object HEAD`
  - `object file ... is empty`
  - `invalid sha1 pointer`
- The corruption affected the local `.git` directory, not the project files.

---

### 2. Git Object Database
- Git stores commits, trees, and blobs inside the `.git/objects` directory.
- Every commit depends on these objects.
- If an object becomes corrupted or empty, Git can no longer reconstruct the repository history.

---

### 3. Diagnosing the Issue
Checked repository integrity using:

```bash
git fsck --full
```

The output reported:
- Corrupted object files
- Invalid `HEAD`
- Invalid branch references
- Missing Git objects

This confirmed the issue was with the repository metadata rather than the working files.

---

### 4. Recovery Process

Preserved the corrupted repository by renaming it:

```bash
mv devops-foundations devops-foundations-broken
```

Cloned a fresh copy from GitHub:

```bash
git clone git@github.com:RaghavN6666/devops-foundations.git
```

Verified the recovered repository:

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

### 5. Local vs Remote Repository
- The local Git repository was corrupted.
- The remote GitHub repository remained healthy.
- Since the latest commits existed on GitHub, cloning a fresh copy restored the repository without losing history.

---

## ❌ Mistakes Made Today
- Initially assumed the problem was caused by Git commands instead of repository corruption.
- Attempted to troubleshoot inside the corrupted repository before confirming that GitHub contained a healthy copy.
- Learned to verify the state of the remote repository before attempting recovery.

---

## 🔑 Key Takeaways
- The `.git` directory contains Git's internal database and can become corrupted independently of project files.
- `git fsck --full` is useful for diagnosing repository corruption.
- A healthy remote repository makes recovery simple through `git clone`.
- Keep the corrupted repository as a backup until recovery is verified.
- Push changes regularly so GitHub can serve as a reliable backup.

---

## 💭 Reflection
Today I encountered my first real Git repository corruption issue. I learned how to diagnose it, recover safely using GitHub, and understand the difference between repository metadata and project files.

---

## 📈 Progress Tracker
- ✅ Linux Fundamentals
- ✅ Git & GitHub
- ✅ Git Repository Recovery
- ✅ Docker Basics
- ✅ Docker Images
- ✅ Docker Containers
- ✅ Dockerfile
- ✅ Container Management
- ✅ Docker Volumes & Bind Mounts
- ✅ Docker Networking
- ✅ Docker Compose
- 🔜 Docker Compose Project (Flask + MySQL)
- 🔜 Docker Best Practices
- 🔜 Docker Troubleshooting Commands
- 🔜 CI/CD with GitHub Actions