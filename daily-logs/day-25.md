# 📅 Daily Learning Log

**Date:** August 4, 2026

## 🎯 Objective

Begin learning Continuous Integration and Continuous Delivery (CI/CD) by understanding the architecture of GitHub Actions, the lifecycle of workflows, and building my first professional CI pipeline from scratch while understanding the reasoning behind every component.

---

# 📚 Topics Covered

## CI/CD Fundamentals

- Why CI/CD Exists
- Continuous Integration (CI)
- Continuous Delivery (CD)
- Manual vs Automated Pipelines

## GitHub Actions

- Workflow
- Events (`on`)
- Jobs
- Steps
- Runners
- GitHub Hosted Runners
- `uses`
- `run`
- `actions/checkout`
- `actions/setup-python`
- `needs`
- Multi-job Workflows

## CI/CD Design Principles

- Ephemeral Environments
- Reproducibility
- Job Isolation
- Pipeline Dependencies
- Exit Codes
- Validation vs Deployment
- Cache
- Artifacts

---

# 🧠 Concepts Learned

## 1. Why CI/CD Exists

Without automation, developers repeatedly perform tasks manually.

```
Edit Code

↓

git add

↓

git commit

↓

git push

↓

Build

↓

Run

↓

Test
```

Problems introduced:

- Human errors
- Inconsistent workflows
- Time-consuming
- Difficult collaboration
- Reduced reliability

CI/CD automates these repetitive processes to improve consistency, speed, and software quality.

---

## 2. Continuous Integration (CI)

Continuous Integration is the practice of integrating code changes into a shared repository frequently.

Every integration automatically performs validation.

```
Developer

↓

Push

↓

Build

↓

Run Tests

↓

Validation
```

Purpose:

- Detect problems early
- Prevent broken integrations
- Maintain a healthy codebase
- Enable efficient collaboration

---

## 3. Continuous Delivery (CD)

Continuous Delivery begins after CI succeeds.

Workflow:

```
CI Passed

↓

Package Application

↓

Build Docker Image

↓

Push Image

↓

Deploy to Staging

↓

Ready for Production
```

Key distinction:

CI validates software.

CD prepares validated software for deployment.

---

## 4. GitHub Actions Architecture

I learned how GitHub automates workflows using hosted runners.

```
Developer

↓

git push

↓

GitHub Repository

↓

Workflow Trigger

↓

GitHub Runner

↓

Execute Workflow

↓

Report Result
```

A major realization today was that after pushing code, my laptop is no longer responsible for executing the pipeline.

GitHub performs everything inside its own infrastructure.

---

## 5. GitHub Runner

A runner is the machine responsible for executing workflow jobs.

Characteristics:

- Temporary
- Fresh environment
- Isolated
- Automatically destroyed after completion

Reasons GitHub creates a fresh runner every workflow:

- Security
- Isolation
- Reliability
- Reproducibility

This closely resembles Docker containers because both are designed as ephemeral environments.

---

## 6. Workflow Structure

A GitHub Actions workflow follows this hierarchy.

```
Workflow

↓

Jobs

↓

Steps

↓

Actions / Commands
```

Each layer has a different responsibility.

Workflow

Entire CI/CD pipeline.

Jobs

Major phases such as:

- Build
- Test
- Deploy

Steps

Individual tasks performed inside each job.

---

## 7. Workflow Triggers

I learned the purpose of:

```yaml
on:
  push:
    branches:
      - main
```

Meaning:

Whenever code is pushed to the main branch, GitHub automatically starts the workflow.

This allows workflows to execute only for selected GitHub events.

---

## 8. Jobs

A job represents one major stage of a workflow.

Example:

```
Workflow

├── Build

├── Test

└── Deploy
```

Each job receives:

- Its own runner
- Its own operating system
- Its own isolated environment

Jobs are not individual commands.

They are collections of related steps.

---

## 9. `runs-on`

I learned why we explicitly specify:

```yaml
runs-on: ubuntu-latest
```

Reasons:

- Compatibility
- Predictable environments
- Reproducibility
- Different projects require different operating systems

This follows the same principle as pinning Docker image versions.

---

## 10. `uses` vs `run`

I learned the distinction between reusable GitHub Actions and shell commands.

### `uses`

Executes a reusable GitHub Action.

Example:

```yaml
- uses: actions/checkout@v4
```

### `run`

Executes shell commands directly on the runner.

Example:

```yaml
- run: pip install -r requirements.txt
```

---

## 11. `actions/checkout`

One of today's most important concepts.

A fresh runner starts empty.

It does not contain:

- app.py
- Dockerfile
- requirements.txt

Therefore:

```yaml
- uses: actions/checkout@v4
```

downloads the repository before any other step can execute.

Without checkout:

```
Runner

↓

python app.py

↓

File Not Found
```

---

## 12. `actions/setup-python`

I learned why Python should still be explicitly configured.

Example:

```yaml
- uses: actions/setup-python@v5
  with:
    python-version: "3.12"
```

Although GitHub runners often already contain Python, explicitly selecting the version guarantees:

- Reproducibility
- Predictable execution
- Version consistency

Again, this follows the same philosophy as Docker image version pinning.

---

## 13. Why CI Should Not Run Flask Servers

Initially I assumed:

```yaml
- run: python app.py
```

would be appropriate.

I learned this is poor CI practice because Flask launches a server that continues waiting indefinitely.

```
Flask

↓

Listening

↓

Waiting...

↓

Waiting...

↓

Workflow Never Finishes
```

Instead, CI should perform validation tasks that terminate naturally.

Example:

```yaml
- run: python -m py_compile app.py
```

This checks syntax, returns an exit code, and exits immediately.

---

## 14. Exit Codes

GitHub Actions relies on Linux exit codes.

```
0
```

Success

```
Non-zero
```

Failure

If a step fails, GitHub immediately stops the workflow and marks it as failed.

---

## 15. My First GitHub Actions Workflow

Today I successfully wrote my first GitHub Actions workflow from memory.

It included:

- Workflow name
- Push trigger
- Ubuntu runner
- Checkout action
- Python setup
- Dependency installation
- Syntax validation

This was the first time I understood every line instead of copying YAML from documentation.

---

## 16. Multi-Job Pipelines

I learned that professional workflows are divided into multiple jobs.

Example:

```
Build

↓

Test

↓

Deploy
```

This improves:

- Modularity
- Reliability
- Maintainability

---

## 17. Job Dependencies (`needs`)

I learned how one job can depend on another.

Example:

```yaml
needs: build
```

Meaning:

The Test job should only execute after the Build job succeeds.

This prevents unnecessary work and protects later stages of the pipeline.

---

## 18. Why Failed Pipelines Stop

One of today's biggest architectural lessons.

If Build fails:

```
Build

❌

↓

Test

(Not Executed)

↓

Deploy

(Not Executed)
```

Reasons:

- Saves compute resources
- Saves time
- Prevents invalid testing
- Prevents broken deployments
- Protects production environments

Every stage acts as a quality gate.

---

## 19. Fresh Runners and Job Isolation

One realization I had today:

Even though Build installs Python packages,

the Test job must install them again.

Reason:

Each job receives a completely new runner.

```
Build Runner

↓

Destroyed

────────────

Test Runner

↓

Fresh Ubuntu
```

Nothing carries over automatically.

This isolation improves:

- Security
- Reliability
- Reproducibility

---

## 20. Cache

I learned that cache is conceptually similar to a Docker volume.

Purpose:

Reuse downloaded dependencies across workflow runs.

Example:

```
Workflow 1

↓

Download Packages

↓

Save Cache

────────────

Workflow 2

↓

Restore Cache

↓

No Download Required
```

Unlike Docker volumes, cache is restored into a fresh runner rather than remaining continuously attached.

---

## 21. Artifacts

Artifacts are files produced during one job that need to be preserved or transferred.

Example:

```
Build

↓

Create report.txt

↓

Upload Artifact

────────────

Test

↓

Download Artifact

↓

Use report.txt
```

If an artifact is **not** uploaded before the runner is destroyed, it is permanently lost because the runner is ephemeral.

Artifacts are stored by GitHub outside the runner and can later be downloaded by other jobs or after the workflow completes.

---

# ❌ Mistakes Made Today

## Mistake 1

Initially mixed Continuous Integration and Continuous Delivery.

### Correction

CI validates software.

CD delivers validated software.

---

## Mistake 2

Initially described jobs as commands.

### Correction

Jobs are major pipeline stages composed of multiple steps.

---

## Mistake 3

Initially believed running Flask inside CI was appropriate.

### Correction

CI should validate software, not host applications.

---

## Mistake 4

Forgot YAML indentation rules while writing my first workflow.

### Correction

YAML hierarchy determines workflow structure.

Proper indentation is critical.

---

## Mistake 5

Initially assumed installed packages from the Build job would automatically be available to the Test job.

### Correction

Each job receives a brand-new isolated runner, requiring dependencies to be installed again unless caching or artifacts are used.

---

# 🔑 Key Takeaways

- CI validates code automatically.
- CD prepares validated software for deployment.
- GitHub Actions runs workflows on temporary runners.
- Every workflow starts with a fresh environment.
- Workflows contain jobs.
- Jobs contain steps.
- `uses` executes reusable GitHub Actions.
- `run` executes shell commands.
- `actions/checkout` downloads the repository.
- `actions/setup-python` guarantees reproducible Python versions.
- CI should validate applications instead of launching long-running servers.
- GitHub Actions relies on Linux exit codes.
- `needs` creates dependencies between jobs.
- Every job is isolated.
- Cache speeds up future workflows by restoring dependencies.
- Artifacts preserve important files outside the runner.

---

# 💭 Reflection

Today's session marked the beginning of my CI/CD journey and significantly changed the way I think about automation. Instead of memorizing GitHub Actions syntax, I focused on understanding why workflows are structured the way they are, how runners execute jobs, and why isolation, reproducibility, and validation are essential in professional software delivery. I also built my first GitHub Actions workflow from scratch and learned how production pipelines are divided into multiple jobs connected through dependencies. Finally, I understood how caching and artifacts solve the challenges introduced by ephemeral runners while maintaining security and reproducibility.

---

# 📈 Progress Tracker

- ✅ Linux Fundamentals
- ✅ Git & GitHub
- ✅ Bash Scripting & Automation Projects
- ✅ Docker Basics
- ✅ Docker Images
- ✅ Docker Containers
- ✅ Dockerfile
- ✅ Container Management
- ✅ Docker Volumes & Bind Mounts
- ✅ Docker Networking
- ✅ Docker Compose
- ✅ Docker Compose Project
- ✅ Docker Best Practices
- ✅ Docker Troubleshooting
- ✅ CI/CD Fundamentals
- ✅ GitHub Actions Architecture
- ✅ GitHub Actions Workflow Structure
- ✅ Multi-Job Workflows
- ✅ Job Dependencies (`needs`)
- ✅ Cache & Artifacts (Concepts)
- 🚀 Next: Advanced GitHub Actions (Environment Variables, Secrets, Caching, Artifacts in Practice & Docker CI Pipeline)