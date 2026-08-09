DAILY LEARNING LOG
Date: August 9, 2026

==================================================
OBJECTIVE
==================================================

Continue and finish the GitHub Actions CI/CD capstone by completing the Docker and deployment architecture.

Today's main focus was understanding why Docker images need persistent storage, how container registries work, why authentication is required for registry operations, and how the exact Docker image can be promoted toward deployment.

The focus remained on understanding the architecture and reasoning rather than memorizing commands.


==================================================
TOPICS COVERED
==================================================

- Docker Jobs in GitHub Actions
- Dockerfiles
- Docker Build Context
- Docker Image Tags
- Git Commit SHA
- Ephemeral Runners
- Container Registries
- Docker Push
- Docker Pull
- Authentication
- Authorization
- Image Versioning
- latest vs Specific Tags
- Immutable Image References
- Container Image Digests
- GitHub Container Registry (GHCR)
- CI/CD Image Promotion
- Deployment Architecture


==================================================
1. DOCKER JOB
==================================================

The next stage after testing is the Docker job:

BUILD
  |
  v
TEST
  |
  v
DOCKER
  |
  v
DEPLOY

The Docker job depends on the successful Test job:

docker:
  needs: test

This means Docker will not execute unless Test succeeds.


==================================================
2. DOCKERFILE
==================================================

A Dockerfile was created for the Flask application.

Example:

FROM python:3.12

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

ENTRYPOINT ["python", "app.py"]

The Dockerfile defines how the application environment and image are created.


==================================================
3. DOCKERFILE INSTRUCTIONS
==================================================

FROM:

Specifies the base image.

Example:

FROM python:3.12

WORKDIR:

Sets the working directory inside the container.

Example:

WORKDIR /app

COPY:

Copies files from the Docker build context into the image.

Example:

COPY requirements.txt .

RUN:

Executes a command during image creation.

Example:

RUN pip install --no-cache-dir -r requirements.txt

ENTRYPOINT:

Defines the main process that runs when the container starts.

Example:

ENTRYPOINT ["python", "app.py"]


==================================================
4. DOCKER BUILD CONTEXT
==================================================

An important Docker concept was learned.

Example:

docker build -t flask-app:${{ github.sha }} test-build

The final argument:

test-build

is the Docker build context.

It is NOT necessarily a Git repository.

In our pipeline it is a directory containing the extracted validated artifact:

test-build/
├── app.py
├── requirements.txt
└── Dockerfile

Therefore Docker uses this directory as the source of files available to the build.


==================================================
5. DOCKER BUILD COMMAND
==================================================

Command:

docker build -t flask-app:${{ github.sha }} test-build

Breakdown:

docker build

Tells Docker to build a container image.

-t

Means tag.

flask-app:${{ github.sha }}

Names the image flask-app and gives it a tag based on the Git commit SHA.

test-build

Specifies the Docker build context.


==================================================
6. GIT COMMIT SHA AS AN IMAGE TAG
==================================================

Instead of using only:

flask-app:latest

we use:

flask-app:${{ github.sha }}

This can produce something similar to:

flask-app:a1b2c3d4

This connects the Docker image to the Git commit that produced it.

Conceptually:

Git Commit
    |
    v
Commit SHA
    |
    v
Docker Image
    |
    v
flask-app:a1b2c3d4


==================================================
7. WHY SPECIFIC IMAGE TAGS ARE USEFUL
==================================================

Discussed the difference between:

flask-app:latest

and:

flask-app:a1b2c3d4

latest is useful when an environment intentionally wants to follow the latest designated image.

However, latest is mutable.

For example:

latest -> Image A

Later:

latest -> Image B

Therefore it is harder to know exactly which image was deployed.

A specific version tag provides better:

- Traceability
- Reproducibility
- Rollback capability
- Auditability
- Deployment consistency


==================================================
8. PROFESSIONAL IMAGE IDENTIFICATION
==================================================

A useful hierarchy was discussed:

latest
    |
    v
Convenient but mutable

Version tag
    |
    v
Better traceability

Git SHA tag
    |
    v
Strong traceability

Image digest
    |
    v
Exact immutable image content

An image can also be referenced using a digest such as:

flask-app@sha256:abc123...


==================================================
9. EPHEMERAL DOCKER RUNNER PROBLEM
==================================================

The Docker job builds an image:

docker build -t flask-app:${{ github.sha }} test-build

The image exists locally inside the Docker runner.

However:

Docker Runner
    |
    +-- Docker image
    |
    +-- Job finishes
    |
    v
Runner destroyed

Therefore the image disappears with the runner.

A later Deploy job receives a fresh runner:

Deploy Runner
    |
    +-- flask-app image does NOT exist locally


==================================================
10. WHY A CONTAINER REGISTRY IS REQUIRED
==================================================

A Container Registry provides persistent storage for Docker images.

Architecture:

Docker Runner
    |
    | docker push
    v
Container Registry
    |
    | docker pull
    v
Deploy Runner

The registry allows the Docker image to survive the destruction of the Docker runner.

Examples of container registries:

- GitHub Container Registry (GHCR)
- Amazon Elastic Container Registry (ECR)
- Docker Hub
- Google Artifact Registry
- Azure Container Registry


==================================================
11. IMAGE VS CONTAINER VS REGISTRY
==================================================

Docker Image:

A packaged application.

Example:

flask-app:a1b2c3d


Container:

A running instance of an image.

Image
  |
  v
docker run
  |
  v
Container


Container Registry:

Persistent storage and distribution system for container images.

Image
  |
  v
docker push
  |
  v
Registry


Simple mental model:

Image = packaged application

Container = running instance

Registry = image warehouse


==================================================
12. DOCKER PUSH
==================================================

After building the image:

docker push

sends the image from the local Docker environment to a remote container registry.

Conceptually:

Docker Runner
    |
    | docker build
    v
Docker Image
    |
    | authenticate
    v
docker push
    |
    v
Container Registry


==================================================
13. DOCKER PULL
==================================================

A deployment environment can retrieve an image from the registry using:

docker pull

Conceptually:

Container Registry
    |
    | docker pull
    v
Deploy Runner
    |
    v
Docker Image
    |
    v
docker run


==================================================
14. AUTHENTICATION VS AUTHORIZATION
==================================================

An important distinction was reinforced.

Authentication asks:

"Who are you?"

Authorization asks:

"What are you allowed to do?"

Conceptually:

Authentication
    |
    v
Identify the requester
    |
    v
Authorization
    |
    v
Determine permissions


==================================================
15. WHY DOCKER PUSH REQUIRES AUTHENTICATION
==================================================

docker build is generally a local operation.

Example:

docker build -t flask-app:a1b2c3d test-build

The runner is creating an image locally.

It is not modifying a remote registry.

Therefore registry authentication is not required merely to build an image.

However:

docker push

modifies a remote registry.

The registry needs to determine:

- Who is requesting the operation?
- Is the requester allowed to push?
- What repository/package can they modify?

Therefore authentication and authorization are required before pushing to a private registry.


==================================================
16. LOCAL BUILD VS REMOTE WRITE
==================================================

Local:

docker build

Means:

Create an image on this machine.

Remote:

docker push

Means:

Write this image to a remote registry.

Therefore:

docker build
    |
    v
Local operation

docker push
    |
    v
Remote authenticated operation


==================================================
17. SECURITY PRINCIPLE
==================================================

Authentication should not be confused with trust itself.

Authentication answers:

"Who are you?"

Authorization answers:

"What are you permitted to do?"

This distinction will become increasingly important when learning:

- AWS IAM
- Kubernetes RBAC
- Cloud permissions
- CI/CD secrets
- Least privilege


==================================================
18. DEPLOYMENT ARCHITECTURE
==================================================

The production-style deployment architecture is:

BUILD
  |
  v
TEST
  |
  v
Docker Build
  |
  v
Security/Test
  |
  v
Docker Push
  |
  v
Container Registry
  |
  v
Docker Pull
  |
  v
DEPLOY


==================================================
19. VALIDATION BEFORE DEPLOYMENT
==================================================

The idea of validating before pulling/deploying was discussed.

The pipeline should use quality gates:

Build:
    Does the application compile?

Test:
    Does the application behave correctly?

Docker:
    Can the application be packaged successfully?

Security/Test:
    Is the image acceptable to release?

Registry:
    Can the validated image be stored?

Deploy:
    Release the validated image.


==================================================
20. IMPORTANT DEPLOYMENT PRINCIPLE
==================================================

The deployment stage should not be the first place where application correctness is discovered.

Instead:

Build
  |
  v
Test
  |
  v
Docker Build
  |
  v
Security / Validation
  |
  v
Registry
  |
  v
Deploy

The goal is to make deployments predictable and boring.

A boring deployment is generally a good deployment because most problems have already been detected earlier in the pipeline.


==================================================
21. CURRENT COMPLETE PRODUCTION-STYLE MODEL
==================================================

The architecture developed so far is:

Developer
    |
    | git push
    v
GitHub Actions
    |
    v
BUILD
    |
    +-- Checkout repository
    +-- Setup Python
    +-- Restore cache
    +-- Install dependencies
    +-- Compile
    +-- Create build artifact
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
    +-- Install artifact dependencies
    +-- Run pytest
    |
    v
DOCKER
    |
    +-- Download artifact
    +-- Extract artifact
    +-- Build Docker image
    +-- Tag with Git SHA
    +-- Authenticate to registry
    +-- Push image
    |
    v
Container Registry
    |
    v
DEPLOY
    |
    +-- Authenticate
    +-- Pull exact image
    +-- Run/deploy image


==================================================
22. BUILD ONCE -> PROMOTE MANY
==================================================

The central CI/CD principle continues to be:

Build once -> validate -> promote the exact output.

For application artifacts:

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


For containerized applications:

Build Image
  |
  v
Push Image
  |
  v
Registry
  |
  v
Test / Scan
  |
  v
Deploy Exact Image


The important goal is:

The image tested and approved should be the image deployed.


==================================================
23. IMPORTANT CORRECTION FROM TODAY
==================================================

A previous explanation described authentication as preventing untrusted developers from finalizing the Docker job.

The more precise understanding is:

Authentication:
    Who are you?

Authorization:
    What are you allowed to do?

The registry uses these mechanisms to control who can push or pull images.

This distinction will become important later with cloud IAM and Kubernetes RBAC.


==================================================
24. CURRENT GITHUB ACTIONS CAPSTONE STATUS
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
DOCKER      [DESIGNED]
    |
    v
REGISTRY    [NEXT]
    |
    v
DEPLOY      [NEXT]


==================================================
25. WHAT WAS SUCCESSFULLY UNDERSTOOD
==================================================

- Dockerfiles
- Docker image creation
- Docker build context
- Docker image tagging
- Git SHA image tags
- latest vs specific tags
- Image immutability concepts
- Ephemeral runner behavior
- Why Docker images disappear with runners
- Container registries
- Docker push
- Docker pull
- Authentication
- Authorization
- Image persistence
- Image distribution
- Deployment architecture
- Build Once -> Promote Many


==================================================
26. WHAT REMAINS
==================================================

Next session:

- GitHub Container Registry authentication
- GHCR permissions
- GITHUB_TOKEN
- Docker login
- Docker push
- Docker pull
- Private registry access
- Registry security
- Least privilege
- Complete real deployment flow
- Final CI/CD capstone review


==================================================
27. KEY TAKEAWAYS
==================================================

- Docker images built inside an ephemeral runner disappear when the runner is destroyed.
- A container registry provides persistent storage for Docker images.
- docker build creates an image locally.
- docker push sends an image to a remote registry.
- docker pull retrieves an image from a registry.
- Authentication identifies the requester.
- Authorization determines what the requester can do.
- latest is convenient but mutable.
- Git SHA tags provide strong traceability.
- Image digests can identify exact immutable image content.
- Production deployments should know exactly which image they are deploying.
- The Docker image should be built once and then promoted.
- A registry bridges isolated CI runners and deployment environments.
- Deploy should consume the validated image rather than rebuild the application.


==================================================
REFLECTION
==================================================

Today I moved from understanding Docker as a tool that builds containers to understanding how Docker fits into a real CI/CD architecture.

I learned that building a Docker image inside a GitHub Actions runner is not enough because the runner is ephemeral. When the job finishes, the runner and its locally stored image disappear.

A container registry solves this problem by providing persistent storage and distribution for Docker images.

I also learned why specific image tags, especially Git commit SHA tags, are preferable to relying only on latest when reproducibility and traceability matter.

Another important concept was the difference between authentication and authorization.

Authentication answers who the requester is, while authorization determines what that identity is allowed to do.

I also connected docker build, docker push, and docker pull to the lifecycle of a production container:

Build -> Authenticate -> Push -> Registry -> Pull -> Deploy


==================================================
PROGRESS
==================================================

GitHub Actions / CI/CD:

Approximately 90% Complete

Completed:

- Jobs
- Runners
- needs
- Checkout
- Python setup
- Dependency installation
- Cache
- Cache keys
- Hashing
- Build validation
- Artifacts
- Artifact upload
- Artifact download
- TAR/GZIP extraction
- Testing
- Pytest
- Dockerfile
- Docker build
- Build context
- Image tags
- Git SHA tagging
- Container registry concepts
- Docker push/pull concepts
- Authentication
- Authorization
- Deployment architecture


Remaining:

- GHCR implementation
- Registry authentication
- Docker push implementation
- Docker pull implementation
- Registry permissions
- Final deployment implementation
- Final end-to-end workflow review


==================================================
NEXT SESSION
==================================================

Resume from:

Docker Image
    |
    v
Authenticate
    |
    v
Docker Push
    |
    v
GitHub Container Registry
    |
    v
Authenticate
    |
    v
Docker Pull
    |
    v
Deploy

The next major topic will be GHCR authentication and permissions, followed by completing the actual registry-backed deployment flow.


==================================================
FINAL STATUS
==================================================

CI/CD PIPELINE:

Developer Push
      |
      v
    BUILD             [DONE]
      |
      v
   ARTIFACT           [DONE]
      |
      v
     TEST             [DONE]
      |
      v
 Docker Build         [DONE/DESIGNED]
      |
      v
Container Registry    [NEXT]
      |
      v
    DEPLOY            [NEXT]

Mastery remains the goal:
understand the architecture, understand the reason behind every component, and then implement it.