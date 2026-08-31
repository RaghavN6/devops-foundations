DAILY LOG — CLOUD / AWS MASTERY

Date: 31 August 2026

TODAY'S FOCUS:
- Stateless applications
- Redis
- Shared session storage
- Sticky sessions
- Caching
- Cache invalidation
- Database bottlenecks
- Connection pooling
- RDS Proxy
- Read Replicas
- Primary database
- Database consistency
- High availability


1. STATELESS APPLICATIONS

Reviewed why application instances should ideally not depend on locally stored user state.

Stateful:

User
 ↓
ALB
 ↓
EC2-A
 ↓
Session stored locally

If EC2-A fails:

EC2-A ❌
 ↓
EC2-B
 ↓
Session may be unavailable ❌


Stateless:

ALB
 ↓
EC2-A / EC2-B
       ↓
Shared external state

Important principle:

"EC2 instances should be disposable; important shared state should not depend on one particular instance."


2. REDIS

Clarified the role of Redis.

Redis acts as a separate, fast shared data store that multiple application instances can access.

Architecture:

             Redis
          Shared Store
          /    |    \
         ↓     ↓     ↓
      EC2-A  EC2-B  EC2-C

Redis can store things such as:

- User sessions
- Cache data
- Counters
- Other temporary/shared application state


3. REDIS AND USER SESSIONS

Example:

User logs in through EC2-A.

EC2-A
 ↓
Redis
 ↓
session_abc123
→ user_id = 123
→ logged_in = true

If EC2-A later fails:

EC2-A ❌
 ↓
ALB → EC2-B
 ↓
EC2-B → Redis
 ↓
Retrieve user's session

The session remains available because it was not stored only inside EC2-A.


4. REDIS IS NOT LITERALLY SHARED MEMORY

Clarified that Redis should not be thought of as memory physically shared between EC2 instances.

More accurate mental model:

EC2-A ──┐
EC2-B ──┼──→ Redis
EC2-C ──┘

Redis is a separate shared data store accessed over the network.


5. STICKY SESSIONS

Reviewed sticky sessions.

Sticky sessions attempt to keep a user's requests going to the same EC2 instance.

Example:

Alice
 ↓
ALB
 ↓
EC2-A

Future requests:

Alice → ALB → EC2-A
Alice → ALB → EC2-A
Alice → ALB → EC2-A

However, sticky sessions do NOT replicate the session.

If:

EC2-A ❌

then a locally stored session can still be lost.

Key distinction:

Sticky Sessions
→ Routing solution

Redis
→ Shared state solution


6. STATEFUL VS STATELESS

Stateful:

User session
 ↓
EC2-A only

Stateless:

User session
 ↓
Shared session store
 ↓
Any healthy EC2 instance

Stateless architecture works particularly well with:

ALB + Auto Scaling


7. AUTO SCALING AND SHARED STATE

Auto Scaling may change the number of instances:

2 instances
 ↓
5 instances
 ↓
3 instances

If sessions are stored locally, adding/removing instances can cause session problems.

With Redis:

             Redis
          /    |    \
       EC2-A EC2-B EC2-C

Instances can be added or removed without taking the shared session state with them.


8. DATABASE VS REDIS

Clarified the different roles.

Database:
→ Durable source of truth for permanent application data.

Redis:
→ Fast shared state/cache/session store.

Example:

Application
   /       \
  ↓         ↓
Database   Redis
  ↓         ↓
Durable    Cache/session


9. PERMANENT USER DATA

Permanent information such as:

user_id = 123
name = Alice
email = alice@example.com

should normally have the primary database as its source of truth.

Redis can optionally cache this data to improve performance.


10. CACHE-ASIDE PATTERN

Reviewed cache-aside behavior.

CACHE HIT:

Application
 ↓
Redis
 ↓
Data found
 ↓
Return cached data

No database query is required for that request, assuming the cached value is valid.


CACHE MISS:

Application
 ↓
Redis
 ↓
No data
 ↓
Database
 ↓
Retrieve data
 ↓
Store in Redis
 ↓
Return data


11. CACHE INVALIDATION

Scenario:

Database:
price = $120

Redis:
price = $100

Problem:

→ Cache inconsistency / stale data.

Possible solutions:

1. TTL

Cached value expires after a defined period.

2. Cache invalidation/update

When the database changes, explicitly remove or update the cached value.

Example:

Price changes
 ↓
Database → $120
 ↓
Invalidate/update Redis
 ↓
Next request gets current value


12. CACHING TRADE-OFF

Caching provides:

→ Better performance
→ Lower database load
→ Faster responses

But introduces:

→ Cache freshness concerns
→ Invalidation requirements
→ Possible stale data

Key principle:

"More performance can introduce a consistency/freshness trade-off."


13. DATABASE BOTTLENECK

Scaling EC2 does not automatically scale the database.

Example:

2 EC2
 ↓
4 EC2
 ↓
8 EC2
 ↓
More application requests
 ↓
More database queries
 ↓
Database becomes overloaded


14. READ REPLICAS

For workloads with many database reads, Read Replicas can distribute read traffic.

Example:

             Primary DB
             /        \
            ↓          ↓
      Read Replica  Read Replica

Primary:
→ Handles writes and can handle reads.

Read Replicas:
→ Primarily handle read queries.

Replication:
→ Keeps replicas updated from the primary.


15. READ REPLICA VS MULTI-AZ

Important distinction:

Multi-AZ:
→ Primarily for high availability and failover.

Read Replica:
→ Primarily for read scaling.

Mental model:

Multi-AZ
→ "What if the primary database fails?"

Read Replica
→ "How can I handle more read traffic?"


16. READ REPLICA LIMITATION

Read Replicas can experience replication lag.

Example:

Primary:
price = $120

Replica:
price = $100

The replica may temporarily be behind the primary.

Therefore:

Read Replicas are useful when some degree of eventual consistency is acceptable.

Critical operations should be carefully routed to the primary.


17. REDIS VS READ REPLICA

Established the distinction:

Repeated/frequently requested data:
→ Redis cache

Large volume of genuinely different database reads:
→ Read Replicas

Example:

10,000 identical product queries:

Redis
→ Avoid repeatedly querying the database.

Many different legitimate database reads:

Read Replicas
→ Distribute database read workload.


18. CRITICAL WRITES

Scenario:

Only one item remains in stock.

Two customers attempt to purchase it.

Critical operation should go to:

Primary Database

because the primary handles the authoritative write/consistency operation.

Example:

Stock = 1

Customer A → Primary → Stock = 0

Customer B → Primary → Out of stock


19. CONNECTION POOLING

Introduced database connection pooling.

Without pooling:

Request
 ↓
Create DB connection
 ↓
Query
 ↓
Destroy connection

Repeated continuously.


With pooling:

Flask
 ↓
Connection Pool
 ┌────┬────┬────┬────┐
 │ C1 │ C2 │ C3 │ C4 │
 └────┴────┴────┴────┘
          ↓
       Database

A request borrows a connection, performs its query, and returns the connection to the pool for reuse.


20. CONNECTION POOL MENTAL MODEL

Connection pooling is more accurately thought of as a:

"Parking lot of reusable database connections"

rather than a queue.

Pool size determines the number of database connections an individual application instance can maintain/use.

Example:

EC2-A
Pool max = 5

→ Up to 5 database connections.


21. CONNECTION SCALING PROBLEM

If:

20 EC2 instances
×
5 connections per instance

Then:

100 possible connections.

If the database only allows:

80 connections

the application can potentially exceed the database's connection capacity.

Connections beyond the available capacity may be rejected, queued, or fail depending on configuration.

Potential symptoms:

- Connection errors
- Timeouts
- Slow requests
- Application errors


22. RDS PROXY

Introduced RDS Proxy as a managed database connection proxy.

Without proxy:

EC2-A ──┐
EC2-B ──┤
EC2-C ──┤
EC2-D ──┤──→ RDS
...     │
EC2-N ──┘


With RDS Proxy:

EC2-A ──┐
EC2-B ──┤
EC2-C ──┤
EC2-D ──┤
         ↓
     RDS Proxy
         ↓
        RDS


23. ROLE OF RDS PROXY

RDS Proxy helps:

→ Manage database connections
→ Reuse connections
→ Reduce connection establishment overhead
→ Handle large numbers of application connections more efficiently
→ Protect the database from excessive connection pressure

Important:

RDS Proxy does NOT magically increase the database's maximum connection limit.

The database still has its own capacity and limits.


24. MULTI-LAYER CONNECTION MODEL

Possible architecture:

Application
 ↓
Connection Pool
 ↓
RDS Proxy
 ↓
RDS

The goal is efficient connection management and preventing the database from being overwhelmed.


25. AUTO SCALING DATABASE CONNECTION RISK

Example:

5 EC2
 ↓
50 EC2

As application capacity increases:

→ More connection pools
→ More potential database connections
→ Greater pressure on database connection limits


26. CURRENT ARCHITECTURE

Current mental model:

                    Internet
                       ↓
                      DNS
                       ↓
                      ALB
                       ↓
                 Target Group
                 ↙          ↘
              EC2-A        EC2-B
                 \          /
                  \        /
                    Redis
                      ↓
                  RDS Proxy
                      ↓
                     RDS


Behind the scenes:

CloudWatch
    ↓
Scaling Policy
    ↓
Auto Scaling Group
    ↓
Launch Template
    ↓
EC2 capacity


27. COMPLETE RESPONSIBILITY MODEL

DNS:
→ Directs users to the application endpoint.

ALB:
→ Distributes incoming traffic.

Target Group:
→ Contains application targets.

Health Check:
→ Determines whether targets are healthy.

CloudWatch:
→ Collects metrics and monitoring information.

Scaling Policy:
→ Determines when/how scaling should occur.

Auto Scaling Group:
→ Maintains desired application capacity.

Launch Template:
→ Blueprint for new EC2 instances.

Redis:
→ Shared fast state/cache/session storage.

RDS Proxy:
→ Manages and reuses database connections.

Primary Database:
→ Durable source of truth and critical writes.

Read Replicas:
→ Scale database reads.

Multi-AZ:
→ Improves availability and failover.


28. IMPORTANT ARCHITECTURAL PRINCIPLE

Different problems require different solutions:

Repeated data requests:
→ Redis

Many database reads:
→ Read Replicas

Too many database connections:
→ Connection pooling / RDS Proxy

Primary database failure:
→ Multi-AZ / failover

More application traffic:
→ ALB + Auto Scaling

User sessions:
→ Shared external session store

Permanent application data:
→ Database


29. KEY LESSON FROM TODAY

Scaling one layer can overload another layer.

Example:

EC2 capacity ↑
        ↓
Application requests ↑
        ↓
Database queries ↑
        ↓
Database load ↑
        ↓
Potential bottleneck


Therefore, DevOps architecture requires thinking about the entire system rather than scaling only one component.


30. PERFORMANCE TODAY

Redis:
GOOD

Stateless Architecture:
GOOD

Sticky Sessions:
GOOD

Caching:
GOOD

Cache Invalidation:
GOOD

Read Replicas:
GOOD

Primary vs Replica:
GOOD

Connection Pooling:
GOOD / REINFORCING

RDS Proxy:
IMPROVING

Database bottleneck reasoning:
GOOD

Architecture reasoning:
STRONG


31. AREAS TO REINFORCE

Continue practicing:

- Database transactions
- Consistency
- Race conditions
- Primary vs Read Replica
- Replication lag
- Connection pooling
- RDS Proxy
- Redis failure/high availability
- Cache invalidation
- Database scaling strategies


32. NEXT SESSION

Next topic:

DATABASE CONSISTENCY + TRANSACTIONS

We will cover:

→ What a transaction is
→ Why transactions matter
→ Race conditions
→ Database locking
→ Atomic operations
→ Isolation
→ Consistency
→ How two users can attempt the same purchase
→ How the database prevents incorrect results

Then we can start connecting these concepts to real AWS architecture and implementation.


FINAL TAKEAWAY

Today's biggest architectural lesson:

"Don't just ask whether the application can scale. Ask whether every dependency can scale with it."

Application:
→ Auto Scaling

Traffic:
→ ALB

Sessions/cache:
→ Redis

Database connections:
→ Connection Pool / RDS Proxy

Database reads:
→ Read Replicas

Database availability:
→ Multi-AZ

Database consistency:
→ Primary + transactions

Good progress today.