DAILY LEARNING LOG
Date: August 10, 2026

==================================================
OBJECTIVE
==================================================

Continue the DevOps CI/CD capstone and move from understanding
container registries conceptually to implementing GitHub Container
Registry (GHCR).

Today's focus:

- Understanding how GHCR works
- Building Docker images locally
- Tagging images for GHCR
- Authenticating to a registry
- Pushing images
- Pulling images
- Running pulled images
- Translating Docker commands into GitHub Actions
- Separating GitHub Actions steps for better debugging
- Understanding GHCR permissions
- Understanding packages: write vs packages: read
- Reinforcing the complete CI/CD mental model


==================================================
1. PRACTICAL CI/CD EXECUTION MODEL
==================================================

We established that the GitHub Actions YAML file is not normally
executed directly inside WSL.

The workflow file is stored inside the GitHub repository:

.github/
└── workflows/
    └── ci-cd.yml

After pushing the repository to GitHub:

WSL
  |
  | git push
  v
GitHub Repository
  |
  v
GitHub Actions
  |
  +-- Build Runner
  +-- Test Runner
  +-- Docker Runner
  └-- Deploy Runner


==================================================
2. WSL VS GITHUB ACTIONS
==================================================

WSL can be used as a local development and testing environment.

Examples:

docker build
docker run
pytest
python -m py_compile
tar
pip install

These commands can be tested locally in WSL.

However, the GitHub Actions YAML itself is executed by GitHub
Actions runners.

The development model is:

WSL
  |
  | develop and test commands
  v
git push
  |
  v
GitHub
  |
  v
GitHub Actions
  |
  v
Actual CI/CD execution


==================================================
3. BASIC CI WORKFLOW
==================================================

We discussed starting with a minimal workflow before adding
advanced components.

Basic workflow:

push
  |
  v
checkout
  |
  v
setup Python
  |
  v
install dependencies
  |
  v
compile
  |
  v
pytest

Example:

name: Flask CI

on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - run: python -m pip install --upgrade pip

      - run: pip install -r requirements.txt

      - run: python -m py_compile app.py

      - run: pytest


==================================================
4. REPOSITORY INSPECTION / DEBUGGING
==================================================

A problem was encountered where the repository or requirements.txt
might not have been found.

Instead of guessing, we introduced repository inspection.

Example:

- name: Inspect repository
  run: |
    echo "Current directory:"
    pwd

    echo ""
    echo "Repository contents:"
    ls -la

    echo ""
    echo "Files in repository:"
    find . -maxdepth 3 -type f | sort

The purpose is to inspect the actual filesystem of the GitHub
Actions runner after checkout.

Mental model:

checkout
  |
  v
Repository files appear on runner
  |
  v
inspect
  |
  v
Understand actual filesystem
  |
  v
Run commands against correct paths


==================================================
5. WHY CHECKOUT MUST HAPPEN FIRST
==================================================

The command:

pip install -r requirements.txt

requires requirements.txt to physically exist on the runner.

Therefore:

actions/checkout@v4
        |
        v
repository files available
        |
        v
requirements.txt exists
        |
        v
pip install -r requirements.txt


Without checkout:

Fresh runner
    |
    v
pip install -r requirements.txt
    |
    v
ERROR: requirements.txt not found


Important distinction:

checkout
    =
get the source files

pip install
    =
install the dependencies


==================================================
6. REQUIREMENTS.TXT
==================================================

requirements.txt is the dependency definition file used by the
Python project.

Example:

Flask==3.1.1
pytest==8.4.1

The command:

pip install -r requirements.txt

means:

Read requirements.txt and install the listed dependencies.

Important distinction:

requirements.txt
    =
dependency definition

~/.cache/pip
    =
cached package data

pip install -r requirements.txt
    =
actual dependency installation


==================================================
7. DOCKER COMMANDS AND GITHUB ACTIONS
==================================================

We established that the underlying Docker commands used manually
in WSL can also be executed inside GitHub Actions.

For example, locally:

docker build ...
docker push ...
docker pull ...
docker run ...

In GitHub Actions:

- name: Build image
  run: |
    docker build ...

- name: Push image
  run: |
    docker push ...

- name: Pull image
  run: |
    docker pull ...

- name: Run container
  run: |
    docker run ...


==================================================
8. PURPOSE OF run: |
==================================================

run: | is YAML multi-line syntax.

It tells GitHub Actions to execute shell commands on the runner.

Example:

- name: Extract artifact
  run: |
    mkdir -p test-build
    tar -xzf dist/flask-app.tar.gz -C test-build

The pipe character does not make Docker work.

It simply allows multiple shell commands to be written as a
multi-line command block.


==================================================
9. SEPARATING GITHUB ACTIONS STEPS
==================================================

We discussed why related operations should be separated into
different steps.

Instead of:

- run: |
    docker --version
    docker build ...
    docker push ...
    docker pull ...

Prefer logical separation:

- name: Verify Docker
  run: docker --version

- name: Build image
  run: docker build ...

- name: Login to GHCR
  run: ...

- name: Push image
  run: docker push ...

This makes the workflow easier to:

- Debug
- Read
- Maintain
- Monitor
- Identify failures


==================================================
10. ERROR MANAGEMENT
==================================================

Separate steps are not required for GitHub Actions to detect
errors.

A normal non-zero exit code causes the step to fail.

However, separate logical steps make the failure much easier
to identify.

Example:

Verify Docker     ✅
Build image       ✅
Login to GHCR     ❌
Push image        ⏭️

This immediately identifies the authentication problem.


==================================================
11. GHCR
==================================================

GHCR means:

GitHub Container Registry.

It is GitHub's container registry for storing and distributing
container images.

The core lifecycle is:

Dockerfile
    |
    v
docker build
    |
    v
Docker Image
    |
    v
docker login
    |
    v
docker push
    |
    v
GHCR
    |
    v
docker pull
    |
    v
docker run
    |
    v
Container


==================================================
12. GHCR IMAGE FORMAT
==================================================

GHCR images use a structure similar to:

ghcr.io/OWNER/IMAGE:TAG

Example:

ghcr.io/john/flask-app:latest

Breakdown:

ghcr.io
    =
GitHub Container Registry

OWNER
    =
GitHub user or organization

IMAGE
    =
container image/package name

TAG
    =
image version/reference


==================================================
13. BUILDING A LOCAL IMAGE
==================================================

Example:

docker build -t flask-app:local .

This creates a local Docker image.

Then:

docker images

can be used to inspect the locally available images.


==================================================
14. TAGGING FOR GHCR
==================================================

The image can be given a GHCR-compatible reference:

docker tag flask-app:local \
  ghcr.io/YOUR_USERNAME/flask-app:latest

Important:

docker tag does not create an entirely separate image.

It gives an existing image another reference/tag.


==================================================
15. AUTHENTICATING TO GHCR
==================================================

To push to GHCR, the Docker client needs to authenticate.

A local workflow can use a GitHub Personal Access Token with
appropriate package permissions.

Conceptually:

Token
  |
  v
docker login
  |
  v
GHCR authentication
  |
  v
docker push permitted


For safer local authentication:

export CR_PAT='YOUR_TOKEN'

Then:

echo "$CR_PAT" | docker login ghcr.io \
  -u YOUR_GITHUB_USERNAME \
  --password-stdin

Important:

Never expose or share the actual token.


==================================================
16. WHY --PASSWORD-STDIN IS USED
==================================================

Instead of:

docker login -u username -p password

we can use:

echo "$CR_PAT" | docker login ghcr.io \
  -u YOUR_GITHUB_USERNAME \
  --password-stdin

The credential is supplied through standard input rather than
being directly included as a command argument.


==================================================
17. DOCKER PUSH
==================================================

After authentication:

docker push ghcr.io/YOUR_USERNAME/flask-app:latest

The image moves from the local Docker environment to GHCR.

Flow:

WSL
  |
  v
Docker Image
  |
  v
docker push
  |
  v
GHCR


==================================================
18. DOCKER PULL
==================================================

To demonstrate that GHCR is actually storing the image:

Remove the local image:

docker rmi ghcr.io/YOUR_USERNAME/flask-app:latest

Then retrieve it:

docker pull ghcr.io/YOUR_USERNAME/flask-app:latest

Flow:

Local Image
    |
    v
docker push
    |
    v
GHCR
    |
    v
docker pull
    |
    v
Local Image again


==================================================
19. DOCKER RUN
==================================================

Once the image is pulled:

docker run --rm \
  ghcr.io/YOUR_USERNAME/flask-app:latest

If Flask listens on port 5000:

docker run --rm \
  -p 5000:5000 \
  ghcr.io/YOUR_USERNAME/flask-app:latest


Lifecycle:

Image
  |
  v
docker run
  |
  v
Container


==================================================
20. CONTAINER REGISTRY MENTAL MODEL
==================================================

Image:

Packaged application.

Container:

Running instance of an image.

Registry:

Persistent storage and distribution system for images.

Simple model:

Image
    =
package

Container
    =
running instance

Registry
    =
warehouse/distribution system


==================================================
21. WHY THE REGISTRY IS REQUIRED
==================================================

GitHub Actions runners are ephemeral.

Docker job:

Docker Runner
    |
    +-- build image
    |
    +-- image exists locally
    |
    +-- push image
    |
    v
Runner destroyed

Without a registry:

Deploy Runner
    |
    +-- image does not exist


With a registry:

Docker Runner
    |
    +-- build
    |
    +-- push
    v
GHCR
    |
    +-- image stored
    |
    v
Deploy Runner
    |
    +-- pull
    |
    +-- run


==================================================
22. AUTHENTICATION VS AUTHORIZATION
==================================================

Authentication:

"Who are you?"

Authorization:

"What are you allowed to do?"

Conceptually:

Authentication
    |
    v
Identify requester
    |
    v
Authorization
    |
    v
Determine permissions


This distinction will become important later with:

- AWS IAM
- Kubernetes RBAC
- Cloud permissions
- CI/CD security
- Least privilege


==================================================
23. GHCR WORKFLOW PERMISSIONS
==================================================

For GitHub Actions to push a container image to GHCR, the workflow
needs appropriate package permissions.

Example:

permissions:
  contents: read
  packages: write

Meaning:

contents: read
    =
read repository contents

packages: write
    =
write/push container packages


==================================================
24. DOCKER JOB PERMISSION
==================================================

The Docker job builds and pushes an image.

Therefore it needs:

packages: write

Example:

docker:
  needs: test
  runs-on: ubuntu-latest

  permissions:
    contents: read
    packages: write


==================================================
25. DEPLOY JOB PERMISSION
==================================================

The Deploy job only needs to retrieve the image.

Therefore it can use:

packages: read

Conceptually:

Docker Job:
    docker push
    ↓
packages: write

Deploy Job:
    docker pull
    ↓
packages: read


==================================================
26. LEAST PRIVILEGE
==================================================

The permission model demonstrates least privilege.

Docker:

packages: write

Deploy:

packages: read

The Deploy job does not need permission to overwrite images in
the registry.

This reduces the impact of a compromised deployment environment.


==================================================
27. GHCR LOGIN IN GITHUB ACTIONS
==================================================

The GitHub Actions workflow can use the automatically provided
GITHUB_TOKEN rather than hard-coding a personal token.

Example:

- name: Login to GHCR
  uses: docker/login-action@v3
  with:
    registry: ghcr.io
    username: ${{ github.actor }}
    password: ${{ secrets.GITHUB_TOKEN }}


==================================================
28. DOCKER JOB WITH GHCR
==================================================

The planned Docker job is:

docker:
  needs: test
  runs-on: ubuntu-latest

  permissions:
    contents: read
    packages: write

  steps:

    Download artifact

        ↓

    Extract artifact

        ↓

    Login to GHCR

        ↓

    Build Docker image

        ↓

    Push Docker image


Conceptual workflow:

Test succeeds
    |
    v
Download artifact
    |
    v
Extract
    |
    v
Authenticate
    |
    v
Docker build
    |
    v
Docker push
    |
    v
GHCR


==================================================
29. DEPLOY JOB WITH GHCR
==================================================

The Deploy job runs on a fresh runner.

Therefore it must retrieve the image from GHCR.

Conceptual workflow:

deploy:
  needs: docker
  runs-on: ubuntu-latest

  permissions:
    packages: read

  steps:

    Login to GHCR

        ↓

    docker pull

        ↓

    docker run / deploy


The image should be the exact image produced by the Docker job.


==================================================
30. COMPLETE CI/CD ARCHITECTURE
==================================================

The pipeline has evolved into:

Developer
    |
    | git push
    v
BUILD
    |
    +-- Checkout
    +-- Setup Python
    +-- Install dependencies
    +-- Compile
    +-- Create artifact
    +-- Upload artifact
    |
    v
GitHub Artifact Storage
    |
    v
TEST
    |
    +-- Download artifact
    +-- Extract artifact
    +-- Install dependencies
    +-- pytest
    |
    v
DOCKER
    |
    +-- Download artifact
    +-- Extract
    +-- Authenticate to GHCR
    +-- Docker build
    +-- Docker push
    |
    v
GHCR
    |
    v
DEPLOY
    |
    +-- Authenticate
    +-- Docker pull
    +-- Run/deploy exact image


==================================================
31. IMPORTANT MEMORY MODEL
==================================================

Instead of memorizing every YAML line, remember what each job
needs to accomplish.

BUILD:

"I need the source."
    ↓
checkout

"I need Python."
    ↓
setup-python

"I need dependencies."
    ↓
pip install

"I want faster dependency retrieval."
    ↓
cache

"I need to preserve my build output."
    ↓
artifact


TEST:

"I need the tests."
    ↓
checkout

"I need the Build output."
    ↓
download artifact

"I need to use it."
    ↓
extract

"I need to verify behavior."
    ↓
pytest


DOCKER:

"I need the validated application."
    ↓
download artifact

"I need the files."
    ↓
extract

"I need registry access."
    ↓
login

"I need a container image."
    ↓
docker build

"I need persistent storage."
    ↓
docker push


DEPLOY:

"I need the registry image."
    ↓
login

"I need the exact image."
    ↓
docker pull

"I need to run/deploy it."
    ↓
docker run / deployment command


==================================================
32. MEMORY WEAKNESS IDENTIFIED
==================================================

It was acknowledged that remembering all of the pipeline contents
is still difficult.

The strategy is NOT to memorize the entire YAML file.

Instead:

Remember the purpose.

Then derive the action from the purpose.

Examples:

"Runner needs repository files"
    ↓
checkout

"Runner needs previous job output"
    ↓
download artifact

"Need faster dependencies"
    ↓
cache

"Need persistent container image storage"
    ↓
registry

"Need to upload image"
    ↓
docker push

"Need image from registry"
    ↓
docker pull

"Job needs permission to modify packages"
    ↓
packages: write

"Job only needs to retrieve packages"
    ↓
packages: read


==================================================
33. KEY DEVOPS DEBUGGING PRINCIPLE
==================================================

When something fails:

Do not immediately guess.

Instead:

ERROR
  |
  v
Inspect environment
  |
  v
Determine actual state
  |
  v
Identify root cause
  |
  v
Fix configuration


Repository inspection commands such as:

pwd
ls -la
find .

are simple but powerful CI debugging tools.


==================================================
34. TODAY'S MOST IMPORTANT CONCEPTS
==================================================

1. WSL is useful for testing commands locally.

2. GitHub Actions executes the workflow YAML on GitHub-hosted
   runners.

3. The same underlying shell/Docker commands can be used locally
   and inside GitHub Actions.

4. run: | executes multi-line shell commands.

5. Separate logical operations into separate GitHub Actions steps.

6. Docker images built inside ephemeral runners disappear with
   those runners unless stored elsewhere.

7. GHCR provides persistent container image storage.

8. docker build creates the local image.

9. docker tag gives the image a registry-compatible reference.

10. docker login authenticates.

11. docker push uploads the image.

12. docker pull retrieves the image.

13. docker run creates a running container from the image.

14. Authentication identifies the requester.

15. Authorization determines what the requester can do.

16. Docker needs packages: write because it pushes images.

17. Deploy only needs packages: read because it pulls images.

18. Least privilege should be used for CI/CD permissions.


==================================================
PROGRESS
==================================================

GitHub Actions / CI/CD:

Approximately 92% Complete

Completed:

- Pipeline architecture
- Jobs
- Runners
- needs
- Checkout
- Python setup
- Dependencies
- requirements.txt
- Cache concepts
- Cache keys
- Hashing
- Build validation
- Artifacts
- Artifact upload/download
- TAR/GZIP extraction
- Test architecture
- Pytest
- Dockerfile
- Docker build
- Docker build context
- Docker image tags
- Git SHA tagging
- Container registries
- GHCR concepts
- Docker push/pull
- Registry authentication concepts
- Authentication vs authorization
- GHCR permissions
- Least privilege
- Registry-backed deployment architecture


Remaining:

- Actually run GHCR locally
- GHCR authentication in WSL
- GHCR push
- GHCR pull
- GHCR authentication in GitHub Actions
- Complete Docker job
- Complete Deploy job
- End-to-end pipeline execution
- Debug real pipeline failures
- Final pipeline reconstruction from memory


==================================================
NEXT SESSION
==================================================

Resume from:

GHCR permissions
      |
      v
Complete GHCR implementation
      |
      v
Docker build
      |
      v
Docker push
      |
      v
GHCR
      |
      v
Deploy login
      |
      v
Docker pull
      |
      v
Docker run / actual deployment


Then perform a memory exercise where the complete pipeline is
reconstructed without looking at previous logs.


==================================================
REFLECTION
==================================================

Today I moved from understanding container registries conceptually
to understanding how GHCR is actually used.

I learned that the same Docker commands tested in WSL can be
executed by GitHub Actions using run:.

I also learned why it is useful to separate commands into multiple
GitHub Actions steps so failures are easier to identify.

The most important new concept was GHCR.

The Docker runner builds the image, authenticates to GHCR, and
pushes the image to the registry. The Deploy runner is fresh, so
it authenticates to GHCR, pulls the image, and then runs/deploys it.

I also learned the security difference between:

packages: write

and:

packages: read

The Docker job needs write permission because it pushes the image.

The Deploy job only needs read permission because it pulls the image.

This reinforced the principle of least privilege.

I also recognized that I do not need to memorize every line of the
pipeline. I should remember what each stage needs to accomplish
and derive the required GitHub Actions feature or shell command
from that purpose.

The mastery goal remains:

Understand the system first.
Then implement it.
Then repeat it until the implementation becomes natural.


==================================================
FINAL PIPELINE MEMORY MAP
==================================================

BUILD
  |
  +-- checkout
  +-- Python
  +-- dependencies
  +-- cache
  +-- validate
  +-- artifact
  |
  v
TEST
  |
  +-- download artifact
  +-- extract
  +-- pytest
  |
  v
DOCKER
  |
  +-- download artifact
  +-- extract
  +-- login
  +-- docker build
  +-- docker push
  |
  v
GHCR
  |
  v
DEPLOY
  |
  +-- login
  +-- docker pull
  +-- docker run / deploy


==================================================
MASTER PRINCIPLE
==================================================

Build once.
Test the build.
Package the validated build.
Store the image persistently.
Retrieve the exact image.
Deploy the validated image.

Do not memorize blindly.

Understand WHY each component exists.