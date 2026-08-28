DAILY LOG — CLOUD / AWS MASTERY

Date: 29 August 2026

Today's Focus:
🗄️ Database Scaling
⚡ Caching
🌐 DNS
⚖️ Application Load Balancers
❤️ Health Checks
🔧 Troubleshooting


1. RETRIEVAL REVIEW

Reviewed the difference between:

Multi-AZ
→ High availability / failover

Read Replica
→ Read scaling

Cache
→ Avoid repeated database queries

Backup
→ Data recovery

Important distinction reinforced:

Primary DB failure
→ Multi-AZ standby / failover

Too many database reads
→ Read Replicas

Repeated identical requests
→ Cache

Accidental data loss
→ Backup


2. READ REPLICAS

Learned that a Read Replica is a separate copy of database data that continuously receives replicated data from the primary.

Concept:

Primary
   ↓
Replication
   ↓
Read Replica
   ↓
Read queries

Writes:
→ Primary

Suitable reads:
→ Read Replicas

Important limitation:
→ Replication lag can mean the replica is temporarily behind the primary.


3. CACHING

Reviewed caching as a mechanism for reducing repeated database queries.

Architecture:

Application
   ↓
Cache
   ├── HIT → Return cached data
   └── MISS → Database
                ↓
             Cache result

Main benefit:
→ Reduce database load
→ Improve response time


4. CACHE STALENESS

Learned that cached data can become outdated.

Example:

Database:
$59.99

Cache:
$49.99

Problem:
→ Cache staleness

Solutions discussed:

Cache invalidation
→ Remove/update stale data

TTL
→ Automatically expire cached data

Trade-off:

Long TTL
→ Fewer DB queries
→ Better cache efficiency
→ Potentially staler data

Short TTL
→ Fresher data
→ More database requests


5. DATABASE FAILOVER

Reviewed the difference between a Multi-AZ standby and a Read Replica.

Multi-AZ:

Primary
   ↓
Standby in another AZ
   ↓
Primary failure
   ↓
Failover
   ↓
Standby becomes primary

Purpose:
→ Availability

Read Replica:

Primary
   ├── Read Replica 1
   ├── Read Replica 2
   └── Read Replica 3

Purpose:
→ Read scaling


6. DNS FUNDAMENTALS

Reviewed DNS from the basics.

DNS maps:

Hostname
   ↓
DNS
   ↓
IP address

Example:

db.example.internal
   ↓
DNS
   ↓
10.0.2.15

Important distinction:

DNS answers:
"Where should I connect?"

DNS does NOT establish the actual network/database connection.


7. DNS AND DATABASE ENDPOINTS

Learned why applications should generally use stable hostnames/endpoints instead of hard-coded database IP addresses.

Poor:

DB_HOST=10.0.2.47

Better:

DB_HOST=database.endpoint

Benefits:
→ Infrastructure can change
→ Failover can occur
→ Application doesn't need a hard-coded database IP


8. DNS HIGH AVAILABILITY

Learned that DNS can return multiple IP addresses.

Example:

app.example.com
   ↓
DNS
 ├── IP-A
 └── IP-B

However:

Multiple DNS records
≠
Automatic health-aware failover

Health checks and appropriate DNS routing/failover configuration are required for DNS-based health-aware routing.

Also compared this with Load Balancers.


9. LOAD BALANCER CONFIGURATION

Learned that AWS provides the Load Balancer technology, but we configure its behavior.

Typical ALB configuration includes:

→ Subnets / AZs
→ Listeners
→ Target Groups
→ Target ports
→ Health checks
→ Security Groups
→ Routing rules
→ TLS certificates when using HTTPS


10. ALB ARCHITECTURE

Concept:

Internet
   ↓
Application Load Balancer
   ↓
Target Group
 ├── EC2-A
 └── EC2-B
       ↓
    Flask :5000

The ALB:
→ Receives traffic
→ Performs health checks
→ Chooses healthy targets
→ Forwards requests


11. FLASK HEALTH ENDPOINT

A Flask application can expose:

/health

Example response:

HTTP 200

The ALB periodically requests the endpoint.

Important distinction:

Flask does NOT proactively send a health signal to the ALB.

Instead:

ALB asks
   ↓
Flask responds
   ↓
ALB evaluates response
   ↓
Healthy / Unhealthy


12. HEALTH CHECK TROUBLESHOOTING

Learned that:

localhost:5000/health → 200

only proves that Flask responds locally.

It does NOT prove:

→ ALB can reach Flask
→ Network routing works
→ Firewall allows traffic
→ Security Group allows traffic
→ Correct target port is configured
→ Health-check path is correct


13. ALB 502 TROUBLESHOOTING

Scenario:

ALB → 502 Bad Gateway

Learned to investigate the ALB-to-backend path:

ALB
 ↓
Target Group
 ↓
EC2
 ↓
Flask
 ↓
Port

Possible issues:

→ Flask not running
→ Wrong listening interface
→ Wrong target port
→ Security Group
→ Network connectivity
→ Application response


14. ALB 503 TROUBLESHOOTING

Scenario:

ALB → 503
All targets unhealthy

Reasoning:

ALB itself may be functioning, but it has no healthy targets available.

Investigate:

→ Health-check configuration
→ Flask
→ Port
→ Listening interface
→ Security Groups
→ Routing
→ NACLs
→ Firewall
→ Application logs


15. HEALTH CHECK CONFIGURATION FAILURE

Example:

ALB checks:

/healthcheck

Flask provides:

/health

Result:

ALB → /healthcheck
      ↓
Flask
      ↓
404 Not Found
      ↓
ALB marks target unhealthy

Fix:

Change ALB health-check path to:

/health


16. TROUBLESHOOTING MINDSET

Major lesson today:

Do not immediately investigate every component.

Instead:

1. Identify what the error proves.
2. Eliminate layers that have already been proven healthy.
3. Follow the actual traffic path.
4. Investigate the most likely remaining layer.

Example:

DNS works
   ↓
ALB works
   ↓
Listener works
   ↓
Target unhealthy
   ↓
Investigate target / health check
   ↓
Don't immediately blame database


17. ARCHITECTURE MODEL DEVELOPED

Current mental model:

Internet
   ↓
DNS
   ↓
ALB
   ↓
Target Group
   ↓
EC2 / Flask
   ↓
Cache
   ↓
Database
   ↓
Read Replicas

With:

Multi-AZ
→ Availability

Auto Scaling
→ Compute capacity

Read Replicas
→ Database read capacity

Cache
→ Reduce repeated database reads

Backup
→ Recovery

Health Checks
→ Detect unhealthy application targets


18. TODAY'S PERFORMANCE

Database concepts:
🟢 Strong

Multi-AZ vs Read Replica:
🟢 Improved significantly

Caching:
🟢 Good

TTL:
🟢 Good

DNS:
🟢 Improving

ALB:
🟢 Good

Health checks:
🟢 Good

Troubleshooting:
🟢 Strong improvement

Main area requiring reinforcement:
🟡 DNS


19. TOMORROW'S PLAN

Start with a short retrieval review:

→ Multi-AZ
→ Read Replica
→ Cache
→ Backup
→ DNS
→ ALB health checks

Then continue:

ALB
   ↓
Auto Scaling
   ↓
Scaling policies
   ↓
CloudWatch metrics
   ↓
Automatic instance replacement
   ↓
Failure scenarios


FINAL NOTE

Today's session was productive.

The biggest improvement was not memorizing more AWS services, but learning to reason through failures layer-by-layer.

Core principle reinforced:

"What exactly has my test proven?"

That question should guide future troubleshooting.