DAILY LEARNING LOG
Date: August 8, 2026

==================================================
OBJECTIVE
==================================================

Continue the GitHub Actions production CI/CD capstone.

Today's focus was designing the CI/CD architecture, understanding job dependencies, learning how artifacts move between isolated runners, creating and extracting build artifacts, and completing the Test job.

The goal remained mastery and understanding rather than memorizing YAML syntax.

==================================================
TOPICS COVERED
==================================================

- Production CI/CD Pipeline Architecture
- Job Dependencies
- needs
- Runner Isolation
- Build -> Test -> Docker -> Deploy
- Fail-Fast Pipeline Design
- GitHub Actions Artifacts
- Artifact Creation
- upload-artifact
- download-artifact
- TAR/GZIP Archives
- YAML Multi-line Syntax
- Cache vs Artifact
- Build Once -> Promote Many
- Testing Build Artifacts
- PYTHONPATH
- Dependency Management
- Professional CI/CD Design

==================================================
1. JOB DEPENDENCIES
==================================================

Learned that needs does more than simply maintain an order.

It creates a dependency between jobs.

Our architecture is:

build
  |
  v
test
  |
  v
docker
  |
  v
deploy

Conceptually:

test:
  needs: build

docker:
  needs: test

deploy:
  needs: docker

Build is the starting job and therefore does not require needs.

==================================================
2. FAILURE PROPAGATION
==================================================

If Build fails:

Build FAILED
  |
  v
Test does not continue
  |
  v
Docker does not continue
  |
  v
Deploy does not continue

If Test fails:

Build SUCCESS
  |
  v
Test FAILED
  |
  v
Docker does not continue
  |
  v
Deploy does not continue

If Docker fails:

Build SUCCESS
  |
  v
Test SUCCESS
  |
  v
Docker FAILED
  |
  v
Deploy does not continue

This creates quality gates throughout the pipeline.

==================================================
3. RUNNER ISOLATION
==================================================

Each GitHub Actions job receives its own runner.

Example:

Build Runner
    |
    v
Runner destroyed

Test Runner
    |
    v
Runner destroyed

Docker Runner
    |
    v
Runner destroyed

Files created on one runner do not automatically exist on another runner.

This is why artifacts are needed when workflow outputs need to move between jobs.

==================================================
4. ARTIFACTS
==================================================

Artifacts preserve important workflow outputs after a runner is destroyed.

Example:

Build Runner
    |
    v
Create flask-app.tar.gz
    |
    v
upload-artifact
    |
    v
GitHub Artifact Storage
    |
    v
download-artifact
    |
    v
Test Runner

Important distinction:

upload-artifact does NOT create the file.

The build commands create the file.

upload-artifact uploads the already-existing file to GitHub Artifact Storage.

==================================================
5. CREATING THE BUILD ARTIFACT
==================================================

Created the build directory:

mkdir -p dist

Created the archive:

tar -czf dist/flask-app.tar.gz app.py requirements.txt Dockerfile

This produces:

dist/
└── flask-app.tar.gz

Then uploaded it:

- uses: actions/upload-artifact@v4
  with:
    name: flask-build
    path: dist/flask-app.tar.gz

==================================================
6. UNDERSTANDING TAR.GZ
==================================================

A .tar.gz file is not technically a ZIP file.

It consists of:

TAR archive
+
GZIP compression

Extraction command:

tar -xzf flask-app.tar.gz

Meaning of the flags:

x = extract
z = gzip
f = file

Therefore:

tar -xzf flask-app.tar.gz

extracts the gzip-compressed TAR archive.

==================================================
7. DOWNLOAD ARTIFACT
==================================================

Build uploads:

- uses: actions/upload-artifact@v4
  with:
    name: flask-build
    path: dist/flask-app.tar.gz

Test downloads:

- uses: actions/download-artifact@v4
  with:
    name: flask-build
    path: dist

The direction is:

upload-artifact:
Runner -> GitHub

download-artifact:
GitHub -> Runner

==================================================
8. DOWNLOAD PATH
==================================================

A mistake was identified when using:

path: dist/flask-app.tar.gz

with download-artifact.

The path represents the destination directory.

Correct:

path: dist

Result:

dist/
└── flask-app.tar.gz

==================================================
9. CACHE VS ARTIFACT
==================================================

CACHE:

Purpose:
Improve performance by reusing dependencies.

Example:

~/.cache/pip

Cache:

Runner
  |
  v
Restore Cache
  |
  v
pip install

ARTIFACT:

Purpose:
Preserve and transfer workflow outputs.

Example:

flask-app.tar.gz

Artifact:

Build
  |
  v
Upload
  |
  v
GitHub Storage
  |
  v
Download
  |
  v
Test

Simple rule:

Cache = reusable dependencies

Artifact = workflow output

==================================================
10. BUILD ONCE -> PROMOTE MANY
==================================================

One of today's most important DevOps principles:

Build once -> validate -> promote the exact artifact.

Instead of:

Build
  |
  v
Test
  |
  v
Rebuild
  |
  v
Deploy

We want:

Build
  |
  v
Artifact
  |
  v
Test
  |
  v
Promote
  |
  v
Deploy

This reduces duplication and improves consistency.

==================================================
11. WHY REBUILDING IS DANGEROUS
==================================================

If Build produces:

app-v1

and Test validates:

app-v1

but Docker independently rebuilds:

app-v2

then:

Tested application != Deployed application

Possible causes of differences include:

- Dependency changes
- Different build environments
- Different tool versions
- External package changes
- Configuration differences
- Build-time differences

Therefore we want the same validated output to continue through the pipeline.

==================================================
12. REAL-WORLD CONTAINER EQUIVALENT
==================================================

For production container pipelines, a common architecture is:

Source
  |
  v
Build Docker Image
  |
  v
Tag with Git SHA
  |
  v
Push to Container Registry
  |
  v
Test / Security Scan
  |
  v
Deploy Exact Image

Example:

my-flask-app:a1b2c3d

The exact image can then be promoted through environments.

This is the container version of:

Build once -> validate -> promote.

==================================================
13. YAML MULTI-LINE PIPE
==================================================

Learned that:

run: |

is YAML syntax for a multi-line string.

Example:

- run: |
    mkdir -p dist
    tar -czf dist/flask-app.tar.gz app.py requirements.txt Dockerfile

The commands inside are then executed by the shell.

Important distinction:

| = YAML syntax

mkdir / tar / etc. = shell commands

==================================================
14. TEST JOB ARCHITECTURE
==================================================

The Test job was designed as:

test:
  needs: build
  runs-on: ubuntu-latest

  steps:

    Checkout Repository

    Setup Python 3.12

    Restore Cache

    Download Build Artifact

    Extract Artifact

    Install Artifact Dependencies

    Run Tests

==================================================
15. WHY CHECKOUT IS STILL NEEDED
==================================================

The artifact contains:

app.py
requirements.txt
Dockerfile

The repository also contains:

tests/
└── test_app.py

The tests were not included in the build artifact.

Therefore checkout provides the Test job with the test suite.

Conceptually:

Checkout
    |
    v
Test files

Artifact
    |
    v
Application being tested

==================================================
16. TESTING THE EXACT ARTIFACT
==================================================

A subtle issue was identified.

If the Test job simply checks out the repository and runs:

pytest

Python could potentially import app.py from the fresh checkout instead of the application inside the Build artifact.

That would violate:

Build once -> test the exact build

Therefore:

PYTHONPATH=test-build pytest

can be used to make Python search the extracted artifact location.

==================================================
17. ARTIFACT DEPENDENCIES
==================================================

The Test job should use:

test-build/requirements.txt

rather than simply relying on:

requirements.txt

from the fresh checkout.

Reason:

The artifact contains the dependency definition that was packaged during the Build stage.

This keeps the Test stage aligned with the exact build output.

==================================================
18. BUILD VALIDATION VS TESTING
==================================================

Build validation:

python -m py_compile app.py

answers:

"Can the Python code be compiled?"

Testing:

pytest

answers:

"Does the application behave correctly?"

Therefore:

Compile != Test

Both are required.

==================================================
19. CURRENT PIPELINE
==================================================

Developer Push
      |
      v
    BUILD
      |
      +-- Checkout
      +-- Setup Python
      +-- Restore Cache
      +-- Install Dependencies
      +-- Compile
      +-- Create Artifact
      +-- Upload Artifact
             |
             v
      GitHub Artifact Storage
             |
             v
     TEST
      |
      +-- Checkout
      +-- Setup Python
      +-- Restore Cache
      +-- Download Artifact
      +-- Extract Artifact
      +-- Install Artifact Dependencies
      +-- Run Tests
             |
             v
          DOCKER
             |
             v
          DEPLOY

==================================================
20. TEST JOB CURRENT DESIGN
==================================================

test:
  needs: build
  runs-on: ubuntu-latest

  steps:
    - uses: actions/checkout@v4

    - uses: actions/setup-python@v5
      with:
        python-version: "3.12"

    - uses: actions/cache@v4
      with:
        path: ~/.cache/pip
        key: ${{ runner.os }}-pip-${{ hashFiles('requirements.txt') }}

    - run: python -m pip install --upgrade pip

    - uses: actions/download-artifact@v4
      with:
        name: flask-build
        path: dist

    - name: Extract build artifact
      run: |
        mkdir -p test-build
        tar -xzf dist/flask-app.tar.gz -C test-build

    - name: Install artifact dependencies
      run: pip install -r test-build/requirements.txt

    - name: Run tests
      run: |
        PYTHONPATH=test-build pytest

==================================================
21. USING PREVIOUS LOGS AS REFERENCE
==================================================

I referenced my previous learning logs while writing the GitHub Actions jobs.

This was recognized as a normal engineering practice rather than something negative.

Professional engineers regularly use:

- Documentation
- Previous implementations
- Internal runbooks
- Personal notes
- Cheat sheets
- Existing repositories

The important distinction is between:

Using notes to remember syntax

and

Copying code without understanding it.

The goal remains understanding the architecture and reasoning behind the implementation.

==================================================
22. MASTERY APPROACH
==================================================

The objective is mastery rather than memorization.

Important principle:

Understand the system first, then use the tools and documentation to implement it.

The goal is to be able to explain:

- Why a job exists
- Why a dependency exists
- Why artifacts are needed
- Why runners are isolated
- Why caches are used
- Why the pipeline stops on failures
- Why we should avoid rebuilding validated outputs

==================================================
23. MISTAKES CORRECTED
==================================================

Mistake 1:

Used upload-artifact inside Test.

Correction:

Build uploads.

Test downloads.

Mistake 2:

Used the artifact filename as the download path.

Correction:

The download path is a directory.

Mistake 3:

Confused creating an artifact with uploading an artifact.

Correction:

Build commands create the file.

upload-artifact stores it.

Mistake 4:

Initially wanted to install dependencies from the checkout.

Correction:

For our architecture, use the requirements file from the extracted artifact.

Mistake 5:

Initially thought Build validation meant the application had already been fully validated.

Correction:

Compilation validates syntax.

Tests validate behavior.

Mistake 6:

Initially thought the .tar.gz artifact was simply a ZIP.

Correction:

.tar.gz is a TAR archive compressed with GZIP.

==================================================
24. KEY TAKEAWAYS
==================================================

- needs expresses job dependencies.
- Each job has an isolated runner.
- Files do not automatically survive between jobs.
- Artifacts preserve workflow outputs.
- upload-artifact uploads an existing file.
- download-artifact retrieves a stored artifact.
- .tar.gz is a TAR archive compressed with GZIP.
- tar -xzf extracts a .tar.gz archive.
- Cache and artifacts solve different problems.
- Cache improves dependency setup speed.
- Artifacts preserve build outputs.
- Build Once -> Promote Many is a fundamental CI/CD principle.
- Build validation and application testing are different.
- PYTHONPATH can control where Python searches for application modules.
- Testing should target the validated build artifact.
- Documentation and personal notes are legitimate engineering tools.
- Understanding is more important than memorizing syntax.

==================================================
TODAY'S PROGRESS
==================================================

GitHub Actions:

85% Complete

Completed:

- Production CI/CD architecture
- Job dependency design
- needs
- Runner isolation
- Artifact lifecycle
- Artifact creation
- Artifact upload
- Artifact download
- Archive extraction
- Build -> Test architecture
- Build Once -> Promote Many
- Test artifact consistency
- Test job design
- Cache vs Artifact
- YAML multi-line syntax

Remaining:

- Docker job
- Docker image architecture
- Docker image tagging
- Container Registry
- Deployment job
- Conditional execution
- Matrix strategies
- Reusable workflows
- Production CI/CD capstone completion

==================================================
CURRENT STATUS
==================================================

BUILD       [COMPLETED]
    |
    v
ARTIFACT    [COMPLETED]
    |
    v
TEST        [COMPLETED]
    |
    v
DOCKER      [NEXT]
    |
    v
DEPLOY      [NEXT]

==================================================
REFLECTION
==================================================

Today's session moved beyond memorizing GitHub Actions syntax and focused on actually designing a CI/CD system.

The most important lesson was understanding runner isolation and how artifacts allow workflow outputs to survive and move between jobs.

I learned that creating an artifact and uploading an artifact are two different operations.

The build process creates the file, while upload-artifact stores that file in GitHub Artifact Storage.

I also learned the important professional principle:

Build Once -> Validate -> Promote the Same Artifact

This prevents different versions from being tested and deployed.

The Test job is now designed around the exact Build artifact rather than independently rebuilding the application.

I also learned that using previous notes and documentation while writing code is normal professional behavior. The goal is not perfect memory; the goal is understanding.

==================================================
NEXT SESSION
==================================================

Continue from:

TEST [COMPLETED]
    |
    v
DOCKER [NEXT]
    |
    v
DEPLOY

Next topics:

- Docker job design
- Converting validated application output into a Docker image
- Docker image tagging
- Container Registry
- Build Once -> Deploy Exact Image
- Deployment simulation
- Completing the GitHub Actions CI/CD capstone