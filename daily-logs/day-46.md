DAILY LOG — AWS / DEVOPS

Date: 1 September 2026

TODAY'S FOCUS:
- Database consistency
- Transactions
- Race conditions
- Database locking
- Atomic operations
- ACID properties
- Isolation levels
- Dirty reads
- Non-repeatable reads
- Deadlocks
- Idempotency
- Payment failure handling
- Reconciliation
- Database bottleneck troubleshooting


1. RACE CONDITIONS

Reviewed how concurrent requests can cause incorrect results.

Example:

Stock = 1

Customer A → reads stock = 1
Customer B → reads stock = 1

Both believe the item is available.

Potential result:

Customer A → purchase succeeds
Customer B → purchase succeeds

2 customers attempting to purchase 1 item creates inconsistent data.

Correct behavior:

Customer A → purchase succeeds
Customer B → "Out of stock"


2. DATABASE LOCKING

Learned that databases can use locks to coordinate concurrent operations.

Example:

Customer A
 ↓
🔒 Lock product row
 ↓
Stock = 1 → 0
 ↓
COMMIT
 ↓
🔓 Release lock

Customer B
 ↓
Waits
 ↓
Sees Stock = 0
 ↓
Purchase rejected


3. APPLICATION MUTEX VS DATABASE LOCK

Important distinction:

Application mutex:
→ Usually protects threads/processes within an application.

Database lock:
→ Protects database data from concurrent database operations.

For database consistency, the database should generally be responsible for coordinating access to its own data.


4. ATOMIC DATABASE OPERATIONS

Reviewed atomic updates.

Example:

UPDATE products
SET stock = stock - 1
WHERE id = 123
AND stock > 0;

This allows the database to perform the check and update as one operation.

Application can check affected rows:

1 row updated:
→ Purchase succeeds.

0 rows updated:
→ Item unavailable.


5. DATABASE TRANSACTIONS

A transaction groups multiple database operations into one logical unit.

Example:

BEGIN
 ↓
Check/update stock
 ↓
Create order
 ↓
Create payment record
 ↓
COMMIT


If something fails:

BEGIN
 ↓
Update stock
 ↓
Create order ❌
 ↓
ROLLBACK


6. ROLLBACK

Rollback reverses database changes made during an unsuccessful transaction.

Example:

Stock = 1

Transaction:
Stock 1 → 0
Create order → FAIL

ROLLBACK

Stock returns to:

1

Important principle:

"Either all required database operations succeed, or the transaction is rolled back."


7. ACID

Learned the four ACID properties:

A — Atomicity
→ All or nothing.

C — Consistency
→ Database remains in a valid state.

I — Isolation
→ Concurrent transactions are controlled so they don't interfere incorrectly.

D — Durability
→ Committed changes survive failures.


8. ATOMICITY

Example:

BEGIN
 ↓
Reduce stock
 ↓
Create order
 ↓
Create payment record ❌
 ↓
ROLLBACK

The stock reduction is undone.

Atomicity:

"All operations succeed, or none of the transaction's database changes should take effect."


9. DURABILITY

If a transaction commits:

Stock:
1 → 0

COMMIT ✅

Then the EC2 application instance crashes.

The committed database change should remain.

EC2:
❌

Database:
Stock = 0 ✅

This is durability.


10. ISOLATION

Isolation controls how concurrent transactions interact.

Example:

Transaction A ──┐
                ├──→ Database
Transaction B ──┘

The database uses isolation mechanisms to prevent unsafe interaction between concurrent transactions.


11. ISOLATION LEVELS

Introduced common isolation levels:

- READ UNCOMMITTED
- READ COMMITTED
- REPEATABLE READ
- SERIALIZABLE

General principle:

Higher isolation:
→ Stronger consistency guarantees
→ Potentially less concurrency/performance

Lower isolation:
→ More concurrency
→ Potentially more anomalies


12. DIRTY READ

Example:

Transaction A:
Balance $100 → $50

But A has not committed.

Transaction B:
Reads balance = $50

Transaction A:
ROLLBACK

Actual balance:
$100

B read data that was never committed.

This is a:

DIRTY READ


13. NON-REPEATABLE READ

Example:

Transaction B:
First read → $50

Another transaction updates:
$50 → $30

Second read by B:
→ $30

The same row produced different committed values during the transaction.

This is a:

NON-REPEATABLE READ


14. DEADLOCK

Example:

Transaction A:
🔒 Row 1
→ waits for Row 2

Transaction B:
🔒 Row 2
→ waits for Row 1

Neither can continue.

This is:

DEADLOCK

Databases can detect deadlocks and typically abort/rollback one transaction so the other can proceed.


15. CONCURRENT DATABASE UPDATES

100 EC2 instances may send concurrent updates to the same database row.

The database does not simply allow uncontrolled modifications.

It uses mechanisms such as:

- Row locks
- Transactions
- Isolation
- Atomic operations
- Concurrency control

The database attempts to provide as much concurrency as possible while preserving correctness.


16. PAYMENT TRANSACTION PROBLEM

Identified an important distributed-systems limitation.

Database rollback can undo database changes.

It cannot automatically undo an external payment.

Example:

Database:
Stock reduced
Order created

Payment provider:
$100 charged

If something fails, database ROLLBACK cannot automatically refund the payment provider.


17. COMPENSATING ACTIONS

A separate refund/compensation mechanism can be designed.

Example:

Payment succeeds
 ↓
Order processing fails
 ↓
Detect failure
 ↓
Call payment provider
 ↓
Refund customer

Important distinction:

Database rollback:
→ Undoes database changes.

Compensation:
→ Reverses an external side effect where possible.


18. EXTERNAL SYSTEM FAILURE

Identified another production failure scenario:

Payment succeeds
 ↓
Flask server crashes
 ↓
Payment was never recorded in database

Database:

Payment = UNKNOWN

Payment provider:

Payment = SUCCESS


19. RECONCILIATION

The application can query the payment provider using the transaction/idempotency identifier.

If payment provider confirms:

SUCCESS

Then update the application's persistent database state:

Payment = PAID

This brings the internal database back into agreement with the external payment system.

Important principle:

"Don't assume an uncertain external operation failed. Verify its actual state and reconcile."


20. IDEMPOTENCY

Learned why idempotency is important for payment systems.

Scenario:

Customer clicks Pay.

Payment succeeds.

Network response is lost.

Application retries.

Without idempotency:

First request:
→ Charge $100

Retry:
→ Charge another $100 ❌

Customer gets charged $200.


21. IDEMPOTENCY KEY

An idempotency key identifies a specific operation.

Example:

Order ID = 5001
Idempotency Key = abc123
Amount = $100

First request:

abc123
→ Charge $100
→ SUCCESS

Retry:

abc123
→ Already processed
→ Do not create another charge
→ Return/reuse the original result


22. IDEMPOTENCY PRINCIPLE

Repeated requests with the same operation identity should not create unintended additional effects.

This is especially useful because distributed systems experience:

- Network failures
- Timeouts
- Server crashes
- Retries
- Duplicate messages


23. PAYMENT + DATABASE ARCHITECTURE

Developed the following conceptual flow:

Customer
 ↓
Application
 ↓
Database transaction
 ├── Reserve/reduce stock
 └── Create order
        ↓
     COMMIT
        ↓
Payment provider
        ↓
Payment succeeds
        ↓
Order confirmed


24. IMPORTANT DISTRIBUTED SYSTEM PRINCIPLE

A database transaction generally cannot atomically control multiple independent systems.

Example:

Database
+
Redis
+
Payment provider
+
Email service

These are separate systems.

A database ROLLBACK cannot automatically rollback actions performed by external services.


25. DATABASE BOTTLENECK TROUBLESHOOTING

Scenario:

EC2 CPU = 30%
RDS CPU = 95%
Redis = healthy
ALB = healthy

Correct approach:

Investigate the database first.

Do NOT automatically add EC2 instances.

Adding more EC2 could generate more database traffic and make the bottleneck worse.


26. CONNECTION BOTTLENECK

Scenario:

RDS CPU = 95%
RDS connections = 90% of maximum
Queries = normal

Potential issue:

Database connection pressure.

Investigate:

- Connection pool sizes
- Active connections
- Idle connections
- Connection leaks
- Connection acquisition timeouts
- RDS Proxy
- Database connection limits


27. DATABASE SCALING MINDSET

Learned:

"Measure → identify bottleneck → choose solution."

Don't automatically respond to every performance problem by adding more application servers.

Find which layer is actually saturated.


28. DATABASE BOTTLENECK DECISION TREE

Repeated identical reads:
→ Redis/cache

Many legitimate database reads:
→ Read Replicas

Too many database connections:
→ Connection pooling / RDS Proxy

Expensive queries:
→ Query/index optimization

Database capacity genuinely insufficient:
→ Scale database appropriately

Primary database failure:
→ Multi-AZ / failover


29. COMPLETE ARCHITECTURE MENTAL MODEL

Users
 ↓
DNS
 ↓
ALB
 ↓
Target Group
 ↓
Auto Scaling EC2
 ↓
 ├── Redis
 │    └── Sessions / Cache
 │
 └── RDS Proxy
       ↓
      RDS
       ├── Primary
       └── Read Replicas


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


30. KEY DEVOPS LESSON

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
Database becomes bottleneck


Therefore:

"Always consider the entire dependency chain when scaling."


31. KEY CONCEPTS MASTERED TODAY

✔️ Race conditions
✔️ Database locks
✔️ Atomic operations
✔️ Transactions
✔️ Rollback
✔️ ACID
✔️ Isolation
✔️ Dirty reads
✔️ Non-repeatable reads
✔️ Deadlocks
✔️ Durability
✔️ Idempotency
✔️ Payment reconciliation
✔️ Compensation/refunds
✔️ Database bottleneck diagnosis
✔️ Connection pressure
✔️ RDS Proxy


32. AREAS TO REINFORCE

Continue practicing:

- Transaction isolation levels
- Locking behavior
- Race-condition prevention
- Idempotent API design
- Distributed transactions
- Reconciliation
- Database failure scenarios
- RDS Proxy
- Read Replica limitations
- Production troubleshooting


33. NEXT SESSION

NEXT TOPIC:

PRODUCTION FAILURE SCENARIO

We will simulate a real incident involving:

Users
 ↓
ALB
 ↓
Auto Scaling
 ↓
Flask
 ↓
Redis
 ↓
RDS Proxy
 ↓
RDS

Then troubleshoot failures step-by-step using:

- Metrics
- Logs
- Health checks
- Network connectivity
- Application behavior
- Redis
- Database connections
- Transactions
- Failover
- Recovery


FINAL TAKEAWAY:

"Don't just design a system for when everything works.

Design it for when something fails halfway through."

Today's biggest lessons:

→ Database = source of truth for persistent data
→ Transactions = protect related database operations
→ Locks/isolation = protect concurrent operations
→ Redis = fast shared state/cache
→ RDS Proxy = connection management
→ Read Replicas = read scaling
→ Idempotency = safe retries
→ Reconciliation = recover from uncertain external state
→ CloudWatch/metrics = identify the actual bottleneck