DAILY LOG — CI/CD MASTERY → CLOUD MASTERY

Today was a major milestone in my DevOps learning journey.

I completed the practical CI/CD pipeline implementation and successfully executed the complete pipeline through GitHub Actions.

I also began transitioning into the next major module: Cloud.


1. COMPLETE CI/CD PIPELINE

The complete pipeline successfully ran:

Git Push
 ↓
BUILD
 ↓
TEST
 ↓
DOCKER
 ↓
GHCR
 ↓
DEPLOY

All jobs successfully completed.

Result:

BUILD       ✅
TEST        ✅
DOCKER      ✅
DEPLOY      ✅


2. PRACTICAL CI/CD IMPLEMENTATION

The pipeline now successfully performs:

BUILD:
- Checks out the repository.
- Sets up Python.
- Installs dependencies.
- Creates an application artifact.
- Uploads the artifact.

TEST:
- Waits for Build using `needs`.
- Downloads the artifact.
- Extracts the artifact.
- Installs dependencies.
- Runs pytest.

DOCKER:
- Waits for Test.
- Downloads the artifact.
- Extracts the Docker build context.
- Logs into GHCR.
- Builds the Docker image.
- Tags the image using the Git SHA.
- Pushes the image to GHCR.

DEPLOY:
- Waits for Docker.
- Uses `packages: read`.
- Logs into GHCR.
- Pulls the exact SHA-tagged image.
- Executes the deployment stage.


3. GHCR

I initially needed to reference my previous daily logs to remember the detailed GHCR implementation.

This showed me that I understand the overall concept but have not completely memorized the exact syntax yet.

Important GHCR concepts reinforced:

Docker → GHCR requires:

packages: write

Deploy → GHCR requires:

packages: read

Authentication uses:

github.actor

and:

secrets.GITHUB_TOKEN

The image is tagged using:

github.sha


4. ARTIFACT VS DOCKER IMAGE

Reinforced the distinction:

GitHub Actions Artifact:

Used to transfer build output between CI jobs.

Docker Image:

The packaged runtime application that is pushed to GHCR and later deployed.

Flow:

Build
 ↓
GitHub Artifact
 ↓
Test / Docker
 ↓
Docker Image
 ↓
GHCR
 ↓
Deploy


5. TROUBLESHOOTING PRACTICE

After successfully running the pipeline, I began deliberately troubleshooting simulated pipeline failures.

This was important because the goal is not only to build pipelines but also to operate and repair them.


6. FAILURE SCENARIO — ARTIFACT NAME

Scenario:

Build uploads:

flask-build-v2

Test tries to download:

flask-build

Diagnosis:

The artifact exists, but Test requests the wrong artifact name.

Correct principle:

The name used by `upload-artifact` must match the name requested by `download-artifact`.

Troubleshooting process:

Job:
TEST

Step:
Artifact download

Expected:
flask-build exists

Actual:
flask-build not found

Likely cause:
Artifact name mismatch

Fix:
Make upload/download names identical.


7. FAILURE SCENARIO — DOCKER BUILD CONTEXT

Scenario:

The artifact was extracted into:

docker-build/

Contents:

docker-build/
├── Dockerfile
├── app.py
└── requirements.txt

But Docker was run with:

docker build -t "$IMAGE_NAME" .

Diagnosis:

`.` means the current directory is the Docker build context.

Docker was therefore looking in the wrong location for the Dockerfile.

Correct command:

docker build -t "$IMAGE_NAME" docker-build

Important Docker concept learned:

`.` does NOT mean "download everything."

It specifies the current directory as the Docker build context.

Docker build context:

The directory whose files are available to Docker during the image build.


8. FAILURE SCENARIO — GHCR PERMISSIONS

Scenario:

Docker build succeeds.

GHCR login succeeds.

Docker push fails:

permission denied

The job had:

packages: read

Diagnosis:

The job is trying to PUSH an image but only has READ permission.

Correct permission:

packages: write

Correct configuration:

permissions:
  contents: read
  packages: write

Reasoning:

Push → write

Pull → read


9. FAILURE SCENARIO — SHA VS LATEST

Scenario:

Docker pushes:

ghcr.io/myuser/flask-cicd:a81f92c

Deploy attempts:

ghcr.io/myuser/flask-cicd:latest

Result:

manifest unknown

Diagnosis:

Deploy is requesting a different image tag from the one Docker pushed.

Correct principle:

The exact image reference pushed by Docker must be the same reference pulled by Deploy.

SHA tagging provides a direct relationship:

Git commit
 ↓
Docker image
 ↓
GHCR
 ↓
Deployment


10. DEPLOYMENT VS APPLICATION HEALTH

A final scenario demonstrated:

Pipeline:

BUILD       ✅
TEST        ✅
DOCKER      ✅
GHCR        ✅
DEPLOY      ✅

But:

curl /health
 ↓
Connection refused

I initially considered:

- app.py
- Dockerfile
- Port mismatch

This was a valid second-level hypothesis.

The deeper issue was that the current deployment step only:

- Pulls the image.
- Prints "Deploying".
- Prints "Deployment successful."

It does NOT actually start a container.

Therefore:

Pipeline success ≠ Application health


11. HEALTHY DEPLOYMENT MODEL

A real deployment should eventually follow:

Pull image
 ↓
Start container
 ↓
Health check
 ↓
HTTP 200
 ↓
Healthy
 ↓
Deployment successful

If the container is running but unhealthy, then investigate:

docker ps
 ↓
docker logs
 ↓
Port mapping
 ↓
Application listening address
 ↓
Application port
 ↓
Health endpoint


12. TROUBLESHOOTING METHODOLOGY

The troubleshooting process has become more systematic.

General process:

Observed failure
 ↓
Identify first failed job
 ↓
Read exact error
 ↓
Identify affected layer
 ↓
Check actual state
 ↓
Compare expected vs actual
 ↓
Form hypothesis
 ↓
Verify hypothesis
 ↓
Fix root cause
 ↓
Re-run


Professional debugging principle:

Diagnose first.
Fix second.

Another important principle:

Use successful stages to eliminate possible causes.

For example:

Docker build successful
+
GHCR login successful
+
Push fails
 ↓
Investigate permissions before rebuilding Docker.


13. CI/CD MODULE STATUS

CI/CD architecture              🟢 Complete
GitHub Actions                  🟢 Complete
Jobs / Steps                    🟢 Complete
needs                            🟢 Complete
Artifacts                       🟢 Complete
Docker                          🟢 Complete
GHCR                            🟢 Complete
Permissions                     🟢 Complete
SHA tagging                     🟢 Complete
Pipeline troubleshooting       🟢 Complete
Real pipeline execution        🟢 Complete
Failure diagnosis              🟢 Complete

Core CI/CD Pipeline Mastery:

🟢 COMPLETE


14. TRANSITION TO CLOUD

The next major module is now:

CLOUD / AWS

The objective has changed.

I originally considered preparing specifically for the AWS Solutions Architect Associate certification.

However, the certification cost is currently too expensive for me.

I therefore decided not to make certification the goal.

Instead, the goal is:

Become exceptionally strong in Cloud/DevOps through practical mastery.


15. NEW CLOUD MASTERY OBJECTIVE

The goal is not:

Study → Memorize → Pass certification

The goal is:

Learn
 ↓
Understand
 ↓
Build
 ↓
Break
 ↓
Troubleshoot
 ↓
Secure
 ↓
Automate
 ↓
Scale
 ↓
Observe
 ↓
Optimize
 ↓
Explain
 ↓
Master


16. CLOUD ROADMAP

Cloud fundamentals
 ↓
Linux + Networking foundations
 ↓
AWS fundamentals
 ↓
IAM + Security
 ↓
VPC + Networking
 ↓
Compute
 ↓
Storage
 ↓
Databases
 ↓
Load Balancing
 ↓
Auto Scaling
 ↓
High Availability
 ↓
Reliability
 ↓
Performance
 ↓
Cost Optimization
 ↓
Containers
 ↓
AWS container platforms
 ↓
Observability
 ↓
Terraform / Infrastructure as Code
 ↓
Production architecture
 ↓
Advanced troubleshooting


17. LONG-TERM DEVOPS ROADMAP

The broader roadmap will eventually cover:

Linux
Networking
Git
CI/CD
Docker
AWS
Terraform
Kubernetes
Security
Observability
Monitoring
Logging
Tracing
Reliability
Scalability
Production architecture
Platform engineering


18. NEW MASTERY STANDARD

For every major technology, I want to reach the point where I can answer:

1. What is it?
2. Why does it exist?
3. How does it work?
4. When should I use it?
5. When should I NOT use it?
6. What are its tradeoffs?
7. How do I implement it?
8. How do I secure it?
9. How do I monitor it?
10. What happens when it breaks?
11. How do I troubleshoot it?
12. How would I explain it to another engineer?


19. LONG-TERM CAREER OBJECTIVE

The goal is not simply to obtain a certificate.

The goal is to become a highly capable DevOps/Cloud engineer with:

- Deep fundamentals.
- Strong systems thinking.
- Practical cloud experience.
- Strong CI/CD skills.
- Infrastructure as Code.
- Container orchestration.
- Security knowledge.
- Observability skills.
- Troubleshooting ability.
- Strong architecture judgment.
- Exceptional portfolio projects.
- Technical interview readiness.


20. PORTFOLIO GOAL

Eventually build several substantial projects rather than many small tutorial projects.

Potential projects:

PROJECT 1:
Production-style CI/CD pipeline

GitHub
 ↓
Tests
 ↓
Artifact
 ↓
Security scanning
 ↓
Docker
 ↓
GHCR
 ↓
Deployment
 ↓
Health verification
 ↓
Rollback


PROJECT 2:
AWS production architecture

Terraform
 ↓
VPC
 ↓
Public/private subnets
 ↓
Load Balancer
 ↓
Application
 ↓
Database


PROJECT 3:
Kubernetes platform

GitHub
 ↓
CI/CD
 ↓
Container Registry
 ↓
Kubernetes
 ↓
Application

With:

- Rolling deployments
- Health probes
- Secrets
- ConfigMaps
- Autoscaling
- Ingress
- Observability
- Rollback


FINAL PROJECT:

A complete production-style cloud system combining:

- AWS
- Terraform
- CI/CD
- Docker
- Kubernetes
- IAM
- Networking
- Security
- Databases
- Monitoring
- Logging
- Tracing
- Alerting
- High availability
- Scaling
- Disaster recovery
- Cost optimization


21. CAREER OBJECTIVE

Long-term objective:

Become strong enough to compete for paid remote DevOps/Cloud internships and eventually professional DevOps roles.

The goal is not merely to claim knowledge.

The goal is to have demonstrable evidence:

Knowledge
 +
Projects
 +
Troubleshooting
 +
Architecture
 +
GitHub
 +
Communication
 =
Strong candidate


22. CURRENT POSITION

CI/CD:

🟢 CORE MASTERY ACHIEVED

Cloud:

🟡 ABOUT TO BEGIN

The next module will be approached from fundamentals rather than immediately memorizing AWS services.

The initial cloud mental model is:

Internet
 ↓
Cloud
 ↓
Networking
 ↓
Compute
 ↓
Application

with security and identity surrounding the infrastructure.


NEXT SESSION:

Begin Cloud Mastery.

Start with:

- What cloud actually solves.
- IaaS vs PaaS vs SaaS.
- Regions.
- Availability Zones.
- High availability.
- VPC fundamentals.
- Networking fundamentals.

Then progressively build toward AWS infrastructure and production architecture.

New standard:

Understand → Build → Break → Troubleshoot → Secure → Optimize → Explain → Master.

The goal is not to rush through the cloud module.

The goal is to become exceptionally strong at it before moving to the next major module.