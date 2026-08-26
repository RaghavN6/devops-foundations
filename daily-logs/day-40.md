DAILY LOG — CLOUD / AWS MASTERY — DAY 2

Date: 27 August 2026

Today's Focus:
☁️ AWS Compute
⚖️ Load Balancing
📈 Auto Scaling
🧠 Stateless Applications
💾 EC2 Storage
🗄️ Database Architecture


1. EC2 / CLOUD COMPUTE

Established the connection between the existing Docker workflow and AWS compute.

Existing workflow:

GitHub
 ↓
Build
 ↓
Test
 ↓
Docker
 ↓
GHCR
 ↓
AWS Compute
 ↓
Application

Conceptual EC2 architecture:

AWS
 ↓
EC2 VM
 ↓
Operating System
 ↓
Docker Engine
 ↓
Container
 ↓
Flask Application

Important distinction:

Users interact with the application.

Users should not normally receive direct access to:

- EC2
- Docker
- The underlying operating system

User interaction:

User
 ↓
HTTPS
 ↓
Load Balancer
 ↓
Application
 ↓
Flask


2. LOAD BALANCING

Clarified that a Load Balancer does NOT prevent instance failure.

Its role is to:

- Distribute traffic
- Perform/consume health checks
- Stop routing new traffic to unhealthy targets
- Send traffic toward healthy targets

Example:

Load Balancer
   ├── EC2-A ❌
   └── EC2-B ✅

Traffic continues toward EC2-B.

Important principle:

Load Balancer
→ distributes traffic

It does NOT:
→ create additional compute capacity


3. HEALTH CHECKS

Learned how health checks interact with load balancing.

Concept:

AZ-A
 └── EC2-A ❌

AZ-B
 └── EC2-B ✅

        ↓
Load Balancer

If EC2-A becomes unhealthy:

Health check
 ↓
EC2-A marked unhealthy
 ↓
Load Balancer stops sending new traffic
 ↓
Healthy targets continue serving traffic

Important refinement:

The Load Balancer does not "transition the AZ."

Instead:

AZ failure
 ↓
Targets become unhealthy
 ↓
Health checks detect failure
 ↓
Load Balancer stops using unhealthy targets
 ↓
Traffic continues through healthy targets in another AZ

This requires healthy capacity to exist elsewhere.


4. INSTANCE FAILURE AND DATA

Discussed what happens when an EC2 instance is terminated.

Important distinction:

Compute failure:
→ application instance disappears

Persistent data:
→ should not depend solely on local instance storage

Poor architecture:

EC2-A
 └── Important application data

EC2-A ❌
 ↓
Data potentially unavailable/lost

Better architecture:

EC2-A ──┐
        ├──→ Persistent data store
EC2-B ──┘

Compute becomes replaceable while important state remains available.


5. PERSISTENT STORAGE VS BACKUP

Established an important distinction:

Persistence ≠ Backup

Persistence:
→ Data can survive compute/resource lifecycle events.

Backup:
→ Data can be recovered after corruption, deletion, or other data-loss events.

Example:

EC2
 ↓
Persistent storage
 ↓
Backup
 ↓
Recovery


6. HORIZONTAL VS VERTICAL SCALING

Introduced two scaling approaches.

Vertical scaling:

Small EC2
 ↓
Larger EC2

Increase the resources of one machine.

Horizontal scaling:

1 EC2
 ↓
3 EC2
 ↓
10 EC2

Increase the number of application instances.

Horizontal scaling can improve:

- Capacity
- Availability
- Resilience

especially when instances are distributed across multiple Availability Zones.


7. AUTO SCALING

Learned that Auto Scaling changes the number of application instances based on configured policies and metrics.

Concept:

Traffic / metric
 ↓
Auto Scaling
 ↓
More EC2 instances
 ↓
Healthy targets
 ↓
Load Balancer
 ↓
Traffic distributed

When demand decreases:

Traffic decreases
 ↓
Scaling policy
 ↓
Instances can be removed
 ↓
Reduced infrastructure usage

This provides elasticity.


8. AUTO SCALING VS LOAD BALANCER

Important distinction:

Load Balancer:

"Where should this request go?"

Auto Scaling:

"How many instances should exist?"

Together:

Traffic
 ↓
Load Balancer
 ↓
EC2 instances
 ↑
Auto Scaling
 ↑
Metrics / demand


9. SCALING METRICS

Discussed that CPU utilization is not automatically the correct scaling metric.

Possible signals include:

- CPU utilization
- Request count
- Request latency
- Queue depth
- Custom application metrics

Correct principle:

Do not simply ask:

"Is CPU high?"

Ask:

"What metric best represents application demand?"


10. INSTANCE FAILURE VS SCALING

Clarified two different events.

Traffic increase:

Demand increases
 ↓
Scaling policy
 ↓
Additional capacity

Instance failure:

Instance becomes unhealthy
 ↓
Health checks / instance health detection
 ↓
Unhealthy instance removed/replaced
 ↓
Desired capacity restored

Auto Scaling is not simply:

"Add an instance when traffic becomes unbearable."

It follows configured scaling and health/replacement behavior.


11. STATELESS APPLICATIONS

Established that applications should ideally be stateless to make horizontal scaling easier.

Problem:

EC2-A
 └── User session stored locally

Request 1 → EC2-A ✅
Request 2 → EC2-B ❌

EC2-B doesn't have the session.

Therefore:

Important application state
should not exist only on one application instance.

Better:

EC2-A ──┐
        ├──→ Shared state
EC2-B ──┘

Potential shared-state systems include:

- Database
- Cache
- Object storage

depending on the data type.


12. WHY STATELESSNESS MATTERS

Stateless design helps with:

- Horizontal scaling
- Load balancing
- Instance replacement
- Availability
- Auto Scaling

Important principle:

Stateless compute
+
Externalized persistent state
=
Easier horizontal scaling


13. EC2 STORAGE — INSTANCE STORE

Instance Store:

- Local/ephemeral storage
- Useful for temporary data
- Useful for scratch space
- Useful for temporary processing
- Should not be treated as the durable source of truth

Mental model:

EC2
 ↓
Instance Store
 ↓
Temporary / scratch data

Data can disappear when the relevant instance/storage lifecycle ends.


14. EC2 STORAGE — EBS

EBS = Elastic Block Store.

Mental model:

EC2
 ↓
EBS Volume
 ↓
Filesystem

EBS acts like a virtual block device / disk attached to EC2.

Potential uses:

- Operating system
- Application filesystem
- Persistent files

Important distinction:

EC2 can disappear while an appropriately configured EBS volume can persist independently.


15. EBS IS NOT A BACKUP

Clarified:

EBS:
→ persistent block storage

EBS Snapshot:
→ backup/recovery mechanism for an EBS volume

Concept:

EBS
 ↓
Snapshot
 ↓
Recovery

Important:

Do not automatically equate:

Backup = EBS Snapshot

The appropriate backup mechanism depends on what data is being backed up.


16. S3

S3 = object storage.

Mental model:

S3 Bucket
 ├── report.pdf
 ├── image.png
 ├── backup.tar.gz
 └── data.json

Applications access S3 through APIs.

Good use cases:

- Images
- Videos
- Documents
- Backups
- Logs
- Static assets
- Build artifacts
- Data files


17. EBS VS S3

EBS:

"I need a disk attached to my compute."

EC2
 ↓
EBS
 ↓
Filesystem

S3:

"I need durable object storage."

Application
 ↓
S3 API
 ↓
Object

They solve different problems.


18. S3 AND STATELESS APPLICATIONS

Poor architecture:

User
 ↓
EC2-A
 ↓
/uploads/profile.jpg

Next request:

User
 ↓
EC2-B
 ↓
File unavailable

Better:

User
 ↓
Application
 ↓
S3
 ↓
profile.jpg

Now:

EC2-A ──┐
        ├──→ S3
EC2-B ──┘

Both application instances can access the same object store.


19. DATABASE VS STORAGE

Clarified that a database is not simply generic storage.

Database provides:

- Structured data
- Queries
- Indexing
- Transactions
- Concurrency control
- Data integrity

Example:

Application
 ├──→ Database
 │      └── Structured application state
 │
 ├──→ S3
 │      └── Objects/files
 │
 └──→ EBS
        └── Block storage


20. STORAGE SCENARIO RESULTS

A:
EC2 operating system disk
→ EBS ✅

B:
Temporary image-processing files
→ Instance Store ✅

C:
User profile pictures
→ S3 ✅

D:
Users, usernames, emails, orders
→ Database ✅

E:
Backups
→ Appropriate backup mechanism

For EBS:
EBS → EBS Snapshot

Important correction:

Backup mechanism depends on the underlying data store.


21. DATABASE NETWORK ARCHITECTURE

Introduced private database architecture.

Typical structure:

Internet
 ↓
Load Balancer
 ↓
Private Application
 ↓
Private Database

Security model:

Internet → Load Balancer       ✅
Load Balancer → Application    ✅
Application → Database         ✅

Internet → Database            ❌


22. DATABASE HIGH AVAILABILITY

Reviewed Multi-AZ database architecture.

Concept:

        Database
        /      \
     AZ-A      AZ-B
    Primary   Standby

If the primary fails:

Primary ❌
 ↓
Standby
 ↓
Failover / promotion
 ↓
Application reconnects

Primary purpose:

Multi-AZ
→ High availability / failover


23. MULTI-AZ VS READ REPLICA

Important distinction introduced.

Multi-AZ:

Purpose:
→ Availability

Question answered:

"What happens if the primary database infrastructure fails?"

Read Replica:

Purpose:
→ Read scaling

Question answered:

"How can we handle a large read workload?"

Mental model:

Multi-AZ
 ↓
Availability

Read Replica
 ↓
Read scaling


24. READ REPLICA ARCHITECTURE

Concept:

Primary DB
 ├──→ Read Replica
 └──→ Read Replica

Potential traffic model:

Writes
 ↓
Primary DB

Reads
 ├──→ Replica 1
 └──→ Replica 2

Read replicas can reduce read pressure on the primary depending on the application's architecture and database capabilities.


25. MULTI-AZ IS NOT A BACKUP

Important distinction:

Multi-AZ
→ Availability / failover

Read Replica
→ Read scaling

Backup
→ Recovery

A replicated/standby database does not replace a proper backup strategy.

Example problem:

Accidental destructive SQL operation

If the change is replicated, replication does not automatically mean the mistake can be undone.

Therefore:

Database
 ↓
Backup
 ↓
Recovery


26. DISASTER RECOVERY

Introduced regional failure as a separate failure domain.

Multi-AZ protects against certain AZ-level failures.

But:

Region A ❌

may require:

Region B
 ↓
Recovery infrastructure

Therefore:

Instance failure
 ↓
Instance redundancy

AZ failure
 ↓
Multi-AZ architecture

Region failure
 ↓
Multi-region disaster recovery strategy


27. CLOUD RELIABILITY HIERARCHY

Established:

Availability:
"Can the application continue operating?"

Persistence:
"Does data survive compute failure?"

Backup:
"Can I recover data after data loss?"

Scaling:
"Can capacity adapt to demand?"

Resilience:
"Can the architecture tolerate failures?"


28. CURRENT CLOUD ARCHITECTURE

The architecture being developed now looks conceptually like:

                         INTERNET
                            │
                            ▼
                     Load Balancer
                       /         \
                      ▼           ▼
                   AZ-A         AZ-B
                Application  Application
                      \         /
                       \       /
                        ▼     ▼
                         Database
                            │
                         Backups

Supporting infrastructure:

VPC
Subnets
Route Tables
Internet Gateway
NAT Gateway
Security Groups
IAM
Monitoring
CI/CD
Container Registry


29. CURRENT CLOUD MASTERY STATUS

Cloud fundamentals              🟢
Regions / AZs                   🟢
Failure domains                 🟢
VPC / Subnets                   🟢
Routing                         🟢
Internet Gateway                🟢
NAT Gateway                     🟢
Security Groups                 🟢
IAM fundamentals                🟢
IAM Roles                       🟢
Trust vs Permissions            🟢
IAM Policy Evaluation           🟢
EC2 fundamentals                🟢
Load Balancing                  🟢
Health Checks                   🟢
Horizontal Scaling              🟢
Auto Scaling                    🟢
Stateless Applications          🟢
Instance Store                  🟢
EBS                             🟢
S3                              🟢
Database architecture           🟢
Multi-AZ                        🟢
Read Replicas                   🟡
Backups                         🟡
Disaster Recovery               🟡


30. NEXT LESSON

Starting point:

Database scenario:

10,000 writes/day
1,000,000 reads/day

Requirement:

Reduce read pressure on the primary database.

Question:

Multi-AZ or Read Replicas?

Then continue with:

- Read replicas
- Database backups
- Database recovery
- Caching
- Database failure scenarios
- Cloud monitoring


FINAL REFLECTION

Today's major progression was connecting compute, scaling, storage, and database architecture into one coherent system.

The key mental model developed today:

              APPLICATION
                   │
        ┌──────────┼──────────┐
        ↓          ↓          ↓
     Compute     State      Objects
        │          │          │
       EC2       DB/cache     S3
        │
   Load Balancer
        │
   Auto Scaling
        │
    Multi-AZ

Core principle:

Compute should be replaceable.
Important state should be externalized.
Traffic should be distributed.
Capacity should scale.
Data should be persistent.
Backups should provide recovery.
Failure should be expected and designed for.

Cloud mastery is progressing from:
"How do I deploy this?"
toward:
"How do I design this so it survives failure?"