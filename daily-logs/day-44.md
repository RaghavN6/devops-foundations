DAILY LOG — CLOUD / AWS MASTERY

Date: 30 August 2026

TODAY'S FOCUS:
- ALB vs Auto Scaling responsibilities
- CloudWatch and scaling policies
- Target Tracking
- Step Scaling
- Launch Templates
- High Availability across AZs
- Stateful vs Stateless applications
- Redis
- Shared Session Storage
- Caching
- Application architecture


1. RETRIEVAL REVIEW

Reviewed the difference between the major components:

ALB
→ Distributes incoming traffic to healthy targets.

Auto Scaling Group
→ Maintains the required number of instances.

CloudWatch
→ Collects and monitors metrics/logs.

Launch Template
→ Blueprint for creating/configuring EC2 instances.

Health Check
→ Determines whether a target is healthy enough to receive traffic.

Target Group
→ Organizes the instances/targets that the ALB can send traffic to.


2. TRAFFIC PATH VS SCALING PATH

Important distinction reinforced.

Traffic path:

User
 ↓
DNS
 ↓
ALB
 ↓
Target Group
 ↓
EC2
 ↓
Flask

Scaling/control path:

CloudWatch
 ↓
Scaling Policy
 ↓
Auto Scaling Group
 ↓
Launch Template
 ↓
EC2

Key lesson:

CloudWatch and Launch Templates are NOT directly in the user's request path.

They operate behind the scenes to monitor and manage the infrastructure.


3. ALB AND UNHEALTHY INSTANCES

If an EC2 instance fails its ALB health check:

EC2
 ↓
Health Check
 ↓
500 / failure
 ↓
ALB marks target unhealthy
 ↓
ALB stops sending normal traffic to that target

Auto Scaling can then replace the unhealthy instance if the ASG is configured to respond to the relevant health checks.

Replacement flow:

Unhealthy EC2
 ↓
Auto Scaling Group
 ↓
Launch Template
 ↓
New EC2
 ↓
Health Check
 ↓
Healthy
 ↓
Target Group
 ↓
ALB
 ↓
Traffic


4. HIGH AVAILABILITY

Reviewed how application availability can be maintained across multiple Availability Zones.

Example:

AZ-A                    AZ-B

EC2-A                   EC2-B
  \                       /
   \                     /
          ALB

If EC2-A fails:

EC2-A ❌
 ↓
ALB stops sending traffic to A
 ↓
EC2-B continues serving users


5. AZ FAILURE

If an entire Availability Zone fails, the application should already have capacity in another AZ.

Example:

AZ-A                    AZ-B

EC2-A                   EC2-B
  ❌                      ✅
                           ↓
                         ALB
                           ↓
                         Users

Key principle:

High availability requires distributing application capacity across AZs rather than relying on one AZ.


6. DATABASE HIGH AVAILABILITY

Reviewed the database side separately from the application side.

Multi-AZ database architecture:

Primary DB
     ↓
Synchronous replication
     ↓
Standby DB in another AZ

If the primary fails:

Primary DB ❌
 ↓
Standby becomes primary
 ↓
Application can reconnect/resume


7. SYNCHRONOUS REPLICATION

Clarified that synchronous replication is not simply "keeping track of usage history."

More accurately:

→ Committed database changes are synchronously replicated to the standby.

Mental model:

Primary
 ↓
Synchronous replication
 ↓
Standby

This helps the standby remain closely synchronized with the primary for failover.


8. STATEFUL APPLICATIONS

A stateful application stores important user state locally on an individual instance.

Example:

User
 ↓
ALB
 ↓
EC2-A
 ↓
Session stored in EC2-A memory

If EC2-A fails:

EC2-A ❌
 ↓
ALB → EC2-B
 ↓
EC2-B does not have EC2-A's local session

Potential result:

→ User may be logged out
→ Session information may be lost


9. STATELESS APPLICATIONS

A stateless application does not depend on important user state being stored only on one EC2 instance.

Instead:

ALB
 ↓
EC2-A / EC2-B
       ↓
Shared external state store

This allows requests to move between instances without losing important session state.


10. REDIS

Introduced Redis as a shared, very fast data store that multiple application instances can access.

Example:

             Redis
          Shared Store
          /    |    \
         ↓     ↓     ↓
      EC2-A  EC2-B  EC2-C

Instead of storing a user's session only on EC2-A:

EC2-A
 ↓
Redis
 ↓
User session

EC2-B can later retrieve the same session:

EC2-B
 ↓
Redis
 ↓
User session


11. WHY REDIS HELPS WITH AUTO SCALING

Auto Scaling makes EC2 instances disposable.

Example:

2 instances
 ↓
3 instances
 ↓
5 instances
 ↓
3 instances

If session data is stored locally, newly created instances may not have the required session information.

With Redis:

             Redis
          /    |    \
       EC2-A EC2-B EC2-C

Any application instance can access the shared session state.

Key principle:

Important shared state should not depend on one disposable EC2 instance.


12. REDIS AND USER SESSIONS

Example:

User logs in through EC2-A.

EC2-A stores:

session_abc123
→ user_id = 123
→ logged_in = true

in Redis.

Later:

User
 ↓
ALB
 ↓
EC2-B
 ↓
Redis
 ↓
session_abc123
 ↓
User remains logged in

If EC2-A crashes:

EC2-A ❌
 ↓
ALB → EC2-B
 ↓
EC2-B → Redis
 ↓
Session still available


13. REDIS VS DATABASE

Important distinction:

Database:
→ Durable source of truth for permanent application data.

Redis:
→ Fast shared store commonly used for sessions, caching, counters, and other temporary/shared state.

Example:

Application
   /       \
  ↓         ↓
Database   Redis
  ↓         ↓
Durable    Cache/session


14. PERMANENT USER DATA

For permanent user information such as:

user_id = 123
name = Alice
email = alice@example.com

the primary database is normally the source of truth.

Example:

Flask
 ↓
Primary Database
 ↓
User profile

Redis can optionally cache that information for faster access.


15. CACHE-ASIDE PATTERN

Reviewed the basic cache-aside pattern.

Cache HIT:

Application
 ↓
Redis
 ↓
Data found
 ↓
Return cached data

No database query is necessary for that request, provided the cached data is valid according to the caching strategy.


Cache MISS:

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


16. STATE VS CACHE VS DATABASE

Important mental model developed:

EC2 local memory:
→ Temporary, instance-specific state.

Redis:
→ Fast shared state / cache / sessions.

Database:
→ Durable source of truth.


17. WHY STATELESS APPLICATIONS MATTER

Stateless application architecture works particularly well with:

ALB
+
Auto Scaling

Example:

             ALB
          /   |   \
       EC2-A EC2-B EC2-C
          \    |    /
             Redis
               ↓
            Database

If one EC2 instance disappears, another instance can continue serving requests because important shared state exists outside the individual instance.


18. FULL ARCHITECTURE DEVELOPED

Current architecture:

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
                  Database

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


19. COMPLETE RESPONSIBILITY MODEL

DNS:
→ Directs users toward the application endpoint.

ALB:
→ Distributes traffic.

Target Group:
→ Contains the targets available to the ALB.

Health Check:
→ Determines whether targets are healthy.

CloudWatch:
→ Observes metrics/logs.

Scaling Policy:
→ Defines when/how scaling should occur.

Auto Scaling Group:
→ Maintains application capacity.

Launch Template:
→ Defines how new EC2 instances should be configured.

Redis:
→ Shared fast state/cache/session storage.

Database:
→ Durable source of truth.

Multi-AZ:
→ Provides high availability/failover.


20. IMPORTANT PRINCIPLE

The application should ideally treat EC2 instances as disposable.

Bad architecture:

User session
 ↓
EC2-A only

Better architecture:

User session
 ↓
Shared session store
 ↓
Any healthy EC2 instance


21. TODAY'S PERFORMANCE

ALB:
GOOD

Auto Scaling:
GOOD

CloudWatch:
GOOD

Target Tracking:
GOOD

Step Scaling:
GOOD

Launch Templates:
GOOD

High Availability:
GOOD

Stateful vs Stateless:
IMPROVING WELL

Redis:
GOOD

Sessions:
GOOD

Caching:
GOOD

Architecture reasoning:
STRONG IMPROVEMENT


22. AREAS TO REINFORCE

Continue reinforcing:

- Traffic path vs scaling/control path
- DNS's role
- Redis vs database
- Stateless vs stateful applications
- Multi-AZ application vs Multi-AZ database
- Health checks
- How ALB and Auto Scaling cooperate


23. KEY CONCEPTS TO REMEMBER

ALB:
→ "Where should traffic go?"

Auto Scaling:
→ "How many instances should exist?"

Launch Template:
→ "How should a new instance be configured?"

CloudWatch:
→ "What is happening?"

Health Check:
→ "Is this target healthy?"

Redis:
→ "Where can all application instances access fast shared state?"

Database:
→ "Where is the durable source of truth?"

Multi-AZ:
→ "How do we survive an Availability Zone failure?"


24. TOMORROW'S PLAN

Start with retrieval questions covering:

→ ALB
→ Target Groups
→ Health Checks
→ Auto Scaling
→ CloudWatch
→ Launch Templates
→ Stateful vs Stateless
→ Redis
→ Multi-AZ

Then move toward practical architecture and implementation:

VPC
 ↓
Subnets / AZs
 ↓
Security Groups
 ↓
ALB
 ↓
Target Group
 ↓
Auto Scaling Group
 ↓
Launch Template
 ↓
Flask
 ↓
Redis
 ↓
Database

After the architecture is solid, begin translating it into Terraform.


FINAL NOTE

Today's biggest achievement was connecting the infrastructure and application layers.

The complete mental model is now:

CloudWatch
 ↓
Scaling Policy
 ↓
Auto Scaling Group
 ↓
Launch Template
 ↓
EC2
 ↓
Health Check
 ↓
Target Group
 ↓
ALB
 ↓
Users

And for application state:

EC2 instances
 ↓
Redis
 ↓
Shared sessions/cache

While permanent data remains in:

Application
 ↓
Database

The key principle reinforced today:

"EC2 instances should be disposable; important shared state should not depend on one particular instance."

Good progress today.