# 📅 Daily Learning Log

**Date:** August 5, 2026

## 🎯 Objective

Continue mastering GitHub Actions by understanding caching and artifacts in depth, learning how GitHub optimizes CI workflows, and understanding how data is preserved between isolated runners.

---

# 📚 Topics Covered

## GitHub Actions Caching

- Cache Fundamentals
- Cache Hit
- Cache Miss
- Cache Keys
- `actions/cache`
- `path`
- `runner.os`
- `hashFiles()`
- Stale Cache Prevention

## GitHub Actions Artifacts

- Artifact Fundamentals
- GitHub Artifact Storage
- `upload-artifact`
- `download-artifact`
- Artifact Lifecycle
- Cache vs Artifacts

---

# 🧠 Concepts Learned

## 1. Why Caching Exists

Every GitHub Actions workflow starts with a completely fresh runner.

Without caching:

```
Push

↓

Fresh Runner

↓

Download Flask

↓

Download Requests

↓

Download MySQL Connector

↓

Run Workflow
```

This process repeats for every workflow, even if nothing changes.

Problems:

- Increased execution time
- Increased bandwidth usage
- Higher cloud costs
- Slower developer feedback

Caching solves this by restoring previously downloaded dependencies.

---

## 2. GitHub Cache

GitHub stores dependency files outside the runner.

Example:

```
Workflow 1

↓

Download Packages

↓

Save Cache

──────────────

Workflow 2

↓

Fresh Runner

↓

Restore Cache

↓

Run Workflow
```

The runner remains fresh, but dependencies no longer require downloading from the internet.

---

## 3. Cache Hit

A cache hit occurs when GitHub successfully finds a cache matching the current cache key.

Example:

```
Fresh Runner

↓

Search Cache

↓

Cache Found

↓

Restore Dependencies

↓

Run Workflow
```

Result:

Dependencies are restored immediately, making workflows significantly faster.

---

## 4. Cache Miss

A cache miss occurs when GitHub cannot find a cache matching the current cache key.

Example:

```
Fresh Runner

↓

Search Cache

↓

No Cache Found

↓

Download Dependencies

↓

Save New Cache
```

This usually happens during:

- First workflow execution
- Dependency updates
- Cache key changes

---

## 5. `actions/cache`

I learned how GitHub provides a reusable cache action.

Example:

```yaml
- uses: actions/cache@v4
```

This action allows workflows to restore and save reusable dependency caches automatically.

---

## 6. `path`

Example:

```yaml
path: ~/.cache/pip
```

Purpose:

Specifies which directory GitHub should cache.

For Python projects this typically contains downloaded package files.

Example:

```
~/.cache/pip

├── Flask
├── Requests
├── PyTest
└── mysql-connector-python
```

Only selected directories are cached rather than the entire repository.

---

## 7. Cache Key

One of today's biggest concepts.

Example:

```yaml
key: ${{ runner.os }}-pip-${{ hashFiles('requirements.txt') }}
```

The cache key uniquely identifies a cache.

It consists of:

- Operating System
- Package Manager
- Hash of Dependency File

This allows GitHub to determine whether an existing cache is still valid.

---

## 8. `runner.os`

I learned why the operating system forms part of the cache key.

Example:

```
Ubuntu

↓

ubuntu-pip-A1B2C3

────────────────

Windows

↓

windows-pip-A1B2C3
```

Separate caches are maintained because different operating systems use different binaries, file structures, and package formats.

This prevents incompatible dependency reuse.

---

## 9. `hashFiles()`

GitHub calculates a hash of important dependency files.

Example:

```
requirements.txt

↓

Hash

↓

A1B2C3
```

If the file changes:

```
requirements.txt

↓

Different Hash

↓

X9Y8Z7
```

GitHub immediately knows the dependency list has changed and creates a new cache.

The hash acts as a fingerprint of the file contents.

---

## 10. Why Hashes Are Used

The hash is not primarily used for security in this context.

Its purpose is to detect changes in dependency files.

Example:

```
requirements.txt

↓

Hash

↓

Unique Fingerprint
```

If the dependency file changes, the hash changes, causing GitHub to create a new cache.

---

## 11. Stale Cache

I learned why poor cache keys are dangerous.

Example:

```
key: my-cache
```

Problems:

- Cannot distinguish operating systems.
- Cannot detect dependency updates.
- May restore outdated packages.
- Can produce inconsistent workflow results.

Good cache keys uniquely identify the dependency state.

---

## 12. Cache Lifecycle

First Workflow

```
Push

↓

Fresh Runner

↓

Checkout Repository

↓

Setup Python

↓

Search Cache

↓

Cache Miss

↓

Install Dependencies

↓

Compile Application

↓

Save Cache

↓

Destroy Runner
```

Second Workflow

```
Push

↓

Fresh Runner

↓

Checkout Repository

↓

Setup Python

↓

Search Cache

↓

Cache Hit

↓

Restore Cache

↓

Install Dependencies (Much Faster)

↓

Compile Application

↓

Destroy Runner
```

---

# 📦 Artifacts

## 13. Why Artifacts Exist

Each GitHub job executes on a different runner.

Example:

```
Build Runner

↓

Create flask-app.zip

↓

Runner Destroyed
```

Without artifacts:

```
Test Runner

↓

Fresh Runner

↓

No flask-app.zip
```

The file would be permanently lost.

---

## 14. GitHub Artifact Storage

Instead of leaving files inside the runner,

GitHub uploads them before the runner is destroyed.

```
Runner

↓

Upload Artifact

↓

GitHub Artifact Storage

↓

Runner Destroyed
```

Future jobs download the artifact when needed.

---

## 15. Uploading Artifacts

Example:

```yaml
- uses: actions/upload-artifact@v4
```

Purpose:

Uploads selected files to GitHub's artifact storage.

Example:

```
dist/flask-app.zip

↓

GitHub Artifact Storage
```

Only explicitly selected files are uploaded.

---

## 16. Downloading Artifacts

Example:

```yaml
- uses: actions/download-artifact@v4
```

Purpose:

Downloads previously uploaded artifacts into a new runner.

Example:

```
GitHub Artifact Storage

↓

Download

↓

Fresh Runner

↓

flask-app.zip
```

This allows isolated jobs to share important outputs.

---

## 17. Artifact Lifecycle

```
Build Runner

↓

Create flask-app.zip

↓

Upload Artifact

↓

GitHub Storage

↓

Runner Destroyed

──────────────

Test Runner

↓

Download Artifact

↓

Use flask-app.zip
```

Artifacts bridge isolated runners.

---

## 18. Cache vs Artifacts

Cache

Purpose:

Improve workflow performance.

Examples:

- Python packages
- npm packages
- Maven dependencies

Artifacts

Purpose:

Preserve workflow outputs.

Examples:

- ZIP files
- Test reports
- Coverage reports
- Logs
- Build outputs

Simple rule:

Cache speeds up workflows.

Artifacts preserve important files.

---

## 19. Professional CI/CD Flow

I now understand the complete lifecycle of a professional GitHub Actions pipeline.

```
Developer Push

↓

Checkout Repository

↓

Restore Cache

↓

Setup Python

↓

Install Dependencies

↓

Compile Application

↓

Upload Artifact

↓

Test Job

↓

Download Artifact

↓

Continue Pipeline
```

---

# ❌ Mistakes Made Today

## Mistake 1

Initially assumed changing `app.py` should create a new cache.

### Correction

The cache key depends on `requirements.txt`.

If dependency files remain unchanged, GitHub restores the existing cache.

---

## Mistake 2

Initially associated hashes only with security.

### Correction

Within GitHub Actions caching, hashes are primarily used to detect dependency changes and determine cache validity.

---

## Mistake 3

Initially believed artifacts represented every file inside the runner.

### Correction

Only files explicitly uploaded using `upload-artifact` become artifacts.

Everything else is deleted when the runner is destroyed.

---

# 🔑 Key Takeaways

- Every GitHub runner starts fresh.
- Cache improves workflow performance.
- Cache Hit restores previously saved dependencies.
- Cache Miss downloads dependencies and creates a new cache.
- `path` specifies which directory should be cached.
- Cache keys uniquely identify dependency states.
- `runner.os` separates caches by operating system.
- `hashFiles()` detects dependency changes.
- Good cache keys prevent stale caches.
- Artifacts preserve important files between isolated jobs.
- `upload-artifact` stores files in GitHub Artifact Storage.
- `download-artifact` restores those files into future jobs.
- Cache improves speed.
- Artifacts preserve outputs.

---

# 💭 Reflection

Today's session helped me understand one of the most important performance optimizations in CI/CD. I learned how GitHub Actions uses cache hits and cache misses to avoid unnecessary downloads while still maintaining fresh, isolated runners. I also understood how cache keys are carefully designed to prevent stale dependencies. Finally, I learned how artifacts solve the challenge of transferring files between isolated jobs by using GitHub Artifact Storage, completing my understanding of how data flows through professional CI/CD pipelines.

---

# 📈 Progress Tracker

- ✅ Linux Fundamentals
- ✅ Git & GitHub
- ✅ Bash Scripting & Automation
- ✅ Docker Fundamentals
- ✅ Docker Compose
- ✅ Docker Best Practices
- ✅ Docker Troubleshooting
- ✅ CI/CD Fundamentals
- ✅ GitHub Actions Architecture
- ✅ Workflow Structure
- ✅ Multi-Job Pipelines
- ✅ Job Dependencies (`needs`)
- ✅ Environment Variables
- ✅ GitHub Secrets
- ✅ Secret Masking
- ✅ Git History Security
- ✅ Cache (Theory & Implementation)
- ✅ Cache Hit & Cache Miss
- ✅ Cache Keys
- ✅ `actions/cache`
- ✅ Artifacts (Theory & Implementation)
- ✅ `upload-artifact`
- ✅ `download-artifact`
- 🚀 Next: Production CI/CD Pipeline (Build, Cache, Test, Docker Build, Artifacts & Deployment Simulation)