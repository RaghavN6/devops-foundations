DAILY LOG — CLOUD / AWS MASTERY

Date: 27 August 2026

Today's Focus:
☁️ Database Architecture
🗄️ Multi-AZ & Read Replicas
⚡ Caching
🌐 DNS Fundamentals
🔧 Troubleshooting


1. DATABASE SCALING

Scenario:
10,000 writes/day
1,000,000 reads/day

Correct solution:
→ Read Replicas

Reason:
The bottleneck is read capacity, so additional read capacity can be provided through replicas.

Key distinction:

Multi-AZ
→ High availability / failover

Read Replicas
→ Read scaling


2. CACHING

Learned that caching can reduce repeated database queries.

Architecture:

User
 ↓
Application
 ↓
Cache
 ├── HIT → Return cached data
 └── MISS → Database
             ↓
          Cache result

Caching is particularly useful when many requests repeatedly request the same relatively stable data.


3. AUTO SCALING VS READ REPLICAS VS CACHE

Established the difference:

Auto Scaling
→ More application compute capacity

Read Replicas
→ More database read capacity

Cache
→ Avoid unnecessary repeated database queries


4. CACHE STALENESS

Identified the problem of stale cached data.

Example:

Database:
$59.99

Cache:
$49.99

The cache is serving outdated information.

Term:
→ Cache staleness

Main mechanisms discussed:

Cache invalidation
→ Remove/update stale data

TTL (Time To Live)
→ Automatically expire cached data after a configured period


5. TTL TRADE-OFF

Long TTL:

Advantages:
→ More cache hits
→ Fewer database queries
→ Lower database load
→ Potentially lower cost

Disadvantage:
→ Data can remain stale for longer

Short TTL:

Advantages:
→ Fresher data

Disadvantage:
→ More cache misses
→ More database requests


6. DATABASE FAILURE / FAILOVER

Architecture:

Primary DB
     ↓
Standby DB
     ↓
Different AZ

If primary fails:

Primary ❌
 ↓
Failover mechanism
 ↓
Standby becomes primary
 ↓
Application reconnects
 ↓
Service restored

Key principle:

Multi-AZ
→ Availability and failover


7. DATABASE TROUBLESHOOTING

After database failover, if Flask still reports connection errors, investigate the application-to-database path.

Recommended investigation:

Application
 ↓
Database connection configuration
 ↓
Network / routing
 ↓
Security Groups / access rules
 ↓
Database endpoint / DNS
 ↓
Database


8. DATABASE ENDPOINTS

Learned why applications should generally use a stable database endpoint rather than hard-coding a specific database IP.

Poor:

DB_HOST=10.0.2.47

Better:

DB_HOST=database.endpoint

Concept:

Application
 ↓
Stable endpoint
 ↓
Current database destination

This makes failover and infrastructure changes easier to handle.


9. DNS

Reviewed the role of DNS.

Basic concept:

Hostname
 ↓
DNS
 ↓
IP address

Important distinction:

DNS answers:
"Where should I connect?"

DNS does NOT itself establish the network/database connection.

Concept:

Application
 ↓
DNS → resolve address
 ↓
Network → establish connectivity
 ↓
Port
 ↓
Database


10. DNS TROUBLESHOOTING

Introduced:

dig
nslookup

Example:

dig db.internal.example

Important error:

NXDOMAIN

This indicates that the queried DNS name/domain does not exist according to the DNS system being queried.

We did not fully complete this section because the material was no longer sticking.


11. TROUBLESHOOTING LAYER MODEL

Reinforced the importance of identifying the failing layer before changing configuration.

Example:

DNS failure:
Hostname cannot resolve
→ Investigate DNS

Network failure:
Hostname resolves but connection times out
→ Investigate routing, security rules, port, etc.

Authentication failure:
Network connection succeeds but login fails
→ Investigate credentials/permissions


12. TODAY'S LEARNING QUALITY

Conceptual understanding:
🟢 Strong earlier in session

Database architecture:
🟢 Good

Multi-AZ vs Read Replica:
🟢 Good

Caching:
🟢 Good

TTL:
🟢 Good

DNS:
🟡 Introduced but not retained well

Troubleshooting:
🟢 Good foundation


13. IMPORTANT OBSERVATION

Toward the end of the session, cognitive fatigue became noticeable.

Decision:
→ Stop rather than force additional material.

This is important for mastery.

The goal is not:

"Finish more topics."

The goal is:

"Actually retain and apply the concepts."


14. TOMORROW'S PLAN

Do NOT immediately continue with advanced DNS.

Start with a short retrieval exercise covering:

Multi-AZ
vs
Read Replica
vs
Cache
vs
Backup

Then rebuild DNS from the fundamentals:

Hostname
 ↓
DNS resolver
 ↓
DNS record
 ↓
IP address
 ↓
Network connection

Then revisit:

NXDOMAIN
DNS resolution failure
Connection timeout
Connection refused
Authentication failure


15. CURRENT POSITION

Cloud conceptual progress:
≈ 40%

Current major strengths:

🟢 Architecture reasoning
🟢 IAM concepts
🟢 Networking fundamentals
🟢 Load balancing
🟢 Auto Scaling
🟢 Stateless architecture
🟢 Storage concepts
🟢 Database availability concepts
🟢 Troubleshooting mindset

Current areas requiring reinforcement:

🟡 DNS
🟡 Database details
🟡 AWS hands-on implementation
🟡 Observability


FINAL NOTE

Today's session was productive even though DNS became difficult toward the end.

Stopping was the correct decision.

Tomorrow's objective is:
RECALL → REBUILD → APPLY

rather than simply adding more material.