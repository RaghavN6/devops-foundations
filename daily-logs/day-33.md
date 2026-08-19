DAILY LOG — CI/CD & DEVOPS

Today I continued learning CI/CD concepts beyond the basic pipeline implementation.

MAIN TOPICS COVERED:

1. GitHub Secrets

I learned that sensitive information such as:

- Database passwords
- AWS access keys
- API tokens
- Credentials

should be stored as Secrets rather than being written directly into the workflow.

Example:

${{ secrets.DATABASE_PASSWORD }}

The important principle is:

Sensitive information → Secrets
Non-sensitive configuration → Variables


2. Variables vs Secrets vs GitHub Context

I learned to distinguish between three types of values.

Secrets:
- DATABASE_PASSWORD
- AWS_ACCESS_KEY
- API_TOKEN

Variables:
- PORT=5000
- ENVIRONMENT=production

GitHub Context:
- ${{ github.repository }}
- ${{ github.sha }}
- ${{ github.actor }}

GitHub provides the context values automatically.


3. Environment-Specific Secrets

I initially thought using one shared password across Development, Staging and Production would avoid duplication.

I learned that this is not the preferred approach for security.

Instead:

Development → DEV credentials
Staging → STAGING credentials
Production → PROD credentials

The purpose is environment isolation and reducing the blast radius if one environment is compromised.

The important principle is:

Separate environments → separate credentials → stronger isolation.


4. GitHub Environments

I learned that GitHub Environments can represent deployment targets such as:

- development
- staging
- production

An environment can contain:

- Environment-specific secrets
- Environment-specific variables
- Deployment protection rules
- Required approvals

A deployment job can specify:

environment: production

This tells GitHub that the job is targeting the Production environment.


5. needs vs Environment Protection

I initially thought that `needs` could be used as a manual approval mechanism.

I corrected this understanding.

`needs` controls job dependencies:

needs: docker

means:

"Do not run this job until the Docker job succeeds."

Environment protection rules control whether a deployment is allowed to proceed.

Therefore:

needs → dependency/order

environment → deployment target

environment protection → approval/security gate


6. Deployment Strategies

I learned several deployment strategies.

Recreate:

v1
↓
Stop v1
↓
Start v2

Simple but can cause downtime.


Rolling Deployment:

v1 v1 v1 v1
↓
v2 v1 v1 v1
↓
v2 v2 v1 v1
↓
v2 v2 v2 v1
↓
v2 v2 v2 v2

Instances are gradually replaced.

Canary Deployment:

A small percentage of users receive the new version first.

Example:

95% → v1
5% → v2

The new version is monitored before increasing traffic.

Blue-Green Deployment:

Blue = current production
Green = new version

Traffic can be switched between them.

If Green fails:

Green → 0%
Blue → 100%

This makes rollback faster because the previous version remains available.


7. Combining Deployment Strategies

I learned that deployment strategies are not necessarily mutually exclusive.

Canary and Rolling can be combined.

Canary controls how much traffic receives the new version.

Rolling controls how instances are gradually replaced.

I also learned that Canary and Blue-Green can be combined.

Example:

Blue = production v1
Green = new v2

Start with:

Blue → 95%
Green → 5%

Monitor Green.

If healthy:

5% → 25% → 50% → 100%

If unhealthy:

Green → 0%
Blue → 100%

This provides gradual exposure and a fast rollback path.


8. Observability

I started learning about observability and why deployment success does not necessarily mean application success.

The three major pillars are:

Logs
Metrics
Traces


Logs:

Tell us what happened.

Example:

ERROR Database connection failed


Metrics:

Provide numerical information about system behaviour.

Examples:

CPU usage
Memory usage
Request rate
Error rate
Latency


Traces:

Show how a request travels through multiple services.

Example:

User
→ Load Balancer
→ Flask API
→ Authentication service
→ Database
→ External API


9. Health Checks

I learned that applications can expose a health endpoint such as:

GET /health

A healthy response might be:

200 OK

This allows infrastructure to determine whether an application is healthy.


10. Liveness vs Readiness

Liveness asks:

"Is the application alive?"

Readiness asks:

"Is the application ready to receive traffic?"

An application could be alive but not ready.

Example:

Application process running
↓
Database unavailable
↓
Application is alive
but
Application is not ready to serve users.


11. Deployment Success vs Application Health

One of the most important lessons today was:

A successful deployment does not necessarily mean a successful release.

A deployment command can succeed while the application has:

- High error rates
- Slow responses
- Database failures
- Memory problems
- Other production issues

Therefore:

Deployment success ≠ Application health


CURRENT LEARNING PATH:

Basic CI/CD Pipeline
↓
Artifacts
↓
Docker
↓
GHCR
↓
Deployment
↓
Secrets
↓
Variables
↓
GitHub Environments
↓
Deployment Approvals
↓
Deployment Strategies
↓
Observability


CURRENT PROJECT STATUS:

Build       — Complete
Test        — Complete
Docker      — Mostly complete
GHCR        — Implementation prepared
Deploy      — Implementation prepared
End-to-end  — Not yet validated

The practical pipeline is currently being kept aside while I expand my understanding of the concepts surrounding production-grade CI/CD.


MAIN TAKEAWAY:

I am beginning to understand that DevOps is not just about creating a pipeline that successfully runs commands.

A mature CI/CD system also needs to consider:

- Security
- Environment isolation
- Credentials
- Deployment approvals
- Rollback
- Availability
- Monitoring
- Application health

The pipeline should not only answer:

"Did the deployment command succeed?"

It should eventually answer:

"Is the new version healthy, and should we continue or roll back?"


WEAK AREAS:

- Exact YAML syntax
- Practical implementation
- End-to-end pipeline execution
- Real GHCR execution
- Deployment implementation
- Monitoring implementation
- Troubleshooting real production failures


NEXT SESSION:

Continue with Observability and Monitoring.

Start by analysing a canary deployment with:

Health check: 200 OK
CPU: 40%
Memory: 55%
Error rate: 12%
Latency: 900ms

Determine whether the deployment should be considered healthy or unhealthy and which metrics should influence the decision.

Then continue learning how monitoring can be connected to automated deployment and rollback.


OVERALL PROGRESS:

The conceptual understanding of CI/CD is becoming much stronger.

I am moving from understanding individual pipeline commands toward understanding how CI/CD systems are designed for security, reliability, controlled releases and safe deployment.

The next goal is to connect:

Deployment
→ Monitoring
→ Health checks
→ Automated decision
→ Rollback