DAILY LOG — AWS / DEVOPS
Date: 2 September 2026

TODAY'S FOCUS:
- Database write vs read scaling
- Read Replicas
- Database routing
- Replication lag
- Write-heavy workloads
- Vertical scaling
- Asynchronous processing
- Amazon SQS
- Queue-based architecture
- Transition from theory to hands-on AWS practice


1. READ VS WRITE SCALING

Reviewed the difference between scaling database reads and writes.

For a read-heavy workload:

    Application
       ↓
    ┌──┴───────────────┐
    ↓                  ↓
Primary            Read Replicas
Writes              Reads

Read Replicas can distribute SELECT workload away from the primary.

Important:
Read Replicas primarily help with READ operations.
They do not directly distribute writes away from the primary.


2. DATABASE ROUTING

Learned that the ALB distributes HTTP/application traffic:

    Users
      ↓
     ALB
      ↓
    Flask instances

The application/database access layer determines where database operations go:

    Flask
     ├── WRITE → Primary
     └── READ  → Read Replica

Therefore, if the primary has high CPU while a replica is barely used, investigate database read/write routing rather than automatically blaming the ALB.


3. REPLICATION LAG

Read Replicas may not immediately contain the newest data from the primary.

Example:

    User updates profile
          ↓
       Primary
          ↓
       COMMIT
          ↓
    Replica catches up

If the user immediately reads from the replica, they could temporarily receive older data.

Therefore:

- Normal reads where slight lag is acceptable → Replica
- Reads requiring immediate consistency → Primary


4. REDIS VS READ REPLICAS

Redis and Read Replicas solve different problems.

Redis:

    Request
       ↓
     Redis
     /   \
   Hit   Miss
    ↓      ↓
 Return   Database

A cache hit can completely avoid a database query.

Read Replica:

    Request
       ↓
    Read Replica
       ↓
    Database query
       ↓
    Response

The replica still processes the query; it simply moves the workload away from the primary.


5. WRITE-HEAVY DATABASE WORKLOAD

Considered:

    95% WRITE
    5% READ

Redis provides limited help because most of the workload is writing.

Read Replicas also cannot directly solve the main write bottleneck.

The primary remains responsible for writes.


6. WRITE SCALING

Possible approaches discussed:

1. Remove unnecessary writes
2. Optimize expensive queries
3. Batch writes
4. Deduplicate writes
5. Debounce frequent updates where appropriate
6. Move non-critical work to asynchronous processing
7. Vertically scale the primary
8. Consider partitioning/sharding at very large scale


7. VERTICAL SCALING

If the primary database genuinely needs more capacity:

    RDS
     ↓
    Larger instance
     ↓
    More CPU / RAM / IOPS

This is vertical scaling.

Important:
Scaling the RDS primary is different from adding Read Replicas.


8. UNNECESSARY WRITES

Learned that before scaling infrastructure, we should investigate whether all writes are necessary.

Example:

    User status = online
    User status = online
    User status = online
    User status = online

Repeated writes may unnecessarily consume database resources.

Possible improvements:

- Deduplicate
- Batch
- Debounce
- Move transient data to Redis
- Move non-critical events to asynchronous processing


9. ASYNCHRONOUS PROCESSING

Identified a useful solution for writes that don't need to happen immediately.

Example:

    Flask
      ↓
    Queue
      ↓
    Workers
      ↓
    Database

The queue temporarily holds work while workers process it at a manageable rate.


10. AMAZON SQS

Learned that Amazon SQS (Simple Queue Service) can act as a message queue.

Instead of:

    Flask
      ↓
    3000 analytics writes/sec
      ↓
    RDS
      ↓
    Overload

We can use:

    Flask
      ↓
    SQS
      ↓
    Workers
      ↓
    RDS

The queue absorbs traffic spikes and decouples the application from database processing.


11. IMPORTANT SQS CONCEPT

SQS doesn't simply hold SQL queries.

Instead, the application sends a message describing the work.

Example:

    {
      "event": "page_view",
      "user_id": 123,
      "page": "/products/42"
    }

A worker receives the message and performs the required processing.


12. CRITICAL VS NON-CRITICAL WORK

Developed the idea of separating synchronous and asynchronous operations.

Example:

                 Flask
                   ↓
            Is this critical?
             /           \
           YES            NO
            ↓              ↓
          RDS             SQS
                           ↓
                        Workers
                           ↓
                          RDS

Critical transaction:
→ Process immediately.

Non-critical analytics:
→ Queue and process later.


13. REALISTIC WRITE WORKLOAD

Example:

    5000 writes/sec

    3000 → Analytics events
    1000 → Normal application data
    1000 → Critical transactions

The analytics workload does not necessarily need to compete with critical transactions for database resources.

A queue can separate the workloads.


14. ARCHITECTURAL TRADE-OFF

Read scaling:

    Relatively straightforward
    ↓
    Primary + Read Replicas

Write scaling:

    More complicated
    ↓
    Multiple independent writers
    ↓
    Consistency / ordering / coordination challenges

Adding multiple write databases can introduce:

- Conflicting writes
- Ordering problems
- Replication lag
- Consistency issues
- Distributed transaction complexity


15. SHARDING

Discussed sharding as a more advanced write-scaling technique.

Example:

    Application
     /    |    \
    ↓     ↓     ↓
   DB1   DB2   DB3

Different portions of the data are distributed between databases.

Sharding can provide greater scalability but adds significant architectural complexity.

Therefore it should generally not be the first solution.


16. PRODUCTION TROUBLESHOOTING MINDSET

Reinforced:

    Measure
      ↓
    Identify bottleneck
      ↓
    Understand root cause
      ↓
    Choose appropriate solution

Do not automatically:

    Problem
      ↓
    Add more EC2 ❌


17. DATABASE BOTTLENECK EXAMPLE

If:

    EC2 CPU = 30%
    RDS CPU = 95%
    Redis = healthy
    ALB = healthy

Adding more EC2 could make the problem worse because more application instances can generate more database traffic.

Instead, investigate RDS first.


18. DATABASE QUERY OPTIMIZATION

Worked through an example where:

    Users table = 10 million rows
    Orders table = 50 million rows

A query for order history was performing a:

    Full table scan ❌

The appropriate response was to investigate the execution plan and add/optimize an appropriate database index.

Result:

    Query:
    1.8 seconds → 80 ms

    RDS CPU:
    96% → 62%

This demonstrated that optimization can sometimes be more effective than simply adding infrastructure.


19. IMPORTANT LESSON

Scaling isn't always the answer.

Sometimes:

    Expensive queries
          ↓
    Query optimization
          ↓
    Cheaper queries
          ↓
    Lower infrastructure load


20. HANDS-ON AWS PRACTICE

We identified that we have now done a substantial amount of architecture/theory practice.

The next phase will be actual hands-on AWS work.

Instead of only answering scenarios, we will build and operate the infrastructure ourselves.


21. HANDS-ON PROJECT PLAN

Phase 1:
    EC2 + Flask

Phase 2:
    Launch Template + Auto Scaling Group

Phase 3:
    Application Load Balancer

Phase 4:
    CloudWatch

Phase 5:
    RDS

Phase 6:
    Redis

Phase 7:
    RDS Proxy

Phase 8:
    SQS

Phase 9:
    Failure testing and recovery


22. TARGET ARCHITECTURE

Eventually build:

    Internet
       ↓
      ALB
       ↓
    Auto Scaling Group
       ↓
    Multiple EC2
       ↓
    Flask
       ↓
    ┌───────────────┐
    ↓               ↓
  Redis          RDS Proxy
                    ↓
                   RDS
              /           \
          Primary       Read Replicas

And for asynchronous work:

    Flask
      ↓
    SQS
      ↓
    Worker
      ↓
    Database


23. NEXT SESSION

IMPORTANT:

We have NOT started the hands-on AWS deployment yet.

Next session starts with:

    PHASE 1 — EC2 + FLASK

We will actually use AWS services rather than continuing only with theoretical scenarios.

Planned progression:

    Launch EC2
       ↓
    Connect to instance
       ↓
    Install/configure Flask
       ↓
    Run application
       ↓
    Test from browser
       ↓
    Understand what is happening at every layer


KEY TAKEAWAYS:

✔️ Read Replicas scale appropriate READ workloads.
✔️ Redis can eliminate repeated database reads through caching.
✔️ The application/database layer handles read/write routing.
✔️ Replication lag can make some reads unsuitable for replicas.
✔️ Write-heavy workloads require different strategies.
✔️ Optimize unnecessary writes before scaling infrastructure.
✔️ Vertical scaling can increase primary database capacity.
✔️ SQS can buffer non-critical asynchronous work.
✔️ Queues decouple application traffic from database processing.
✔️ Sharding can scale writes but introduces significant complexity.
✔️ Always identify the bottleneck before choosing a scaling strategy.
✔️ We are now moving from theory into hands-on AWS practice.

NEXT SESSION:
EC2 + Flask — BUILD IT FOR REAL 🚀