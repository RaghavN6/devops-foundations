DAILY LOG — CLOUD / AWS MASTERY

Date: 29 August 2026

TODAY'S FOCUS:
- Application Load Balancer (ALB)
- Auto Scaling
- CloudWatch
- Launch Templates
- Health Checks
- Target Tracking
- Step Scaling
- Proactive vs Reactive Scaling
- High-Availability Architecture


1. RETRIEVAL REVIEW

Reviewed the difference between:

Multi-AZ
→ High availability / failover

Read Replica
→ Database read scaling

Cache
→ Reduce repeated database queries

Backup
→ Data recovery

Important distinction:

Primary DB failure
→ Multi-AZ standby / failover

Too many database reads
→ Read Replica

Repeated identical requests
→ Cache

Accidental data loss
→ Backup


2. LOAD BALANCER VS AUTO SCALING

ALB:
→ Distributes incoming traffic between healthy targets.

Auto Scaling Group:
→ Adjusts the number of instances based on scaling policies.

Key mental model:

ALB = "Where should traffic go?"

Auto Scaling = "How many instances should exist?"


3. ALB HEALTH CHECKS

The ALB periodically checks whether targets are healthy.

Example:

ALB
 ↓
GET /health
 ↓
Flask
 ↓
200 OK
 ↓
Target = Healthy

If the application returns an error or times out:

ALB
 ↓
GET /health
 ↓
500 / timeout
 ↓
Target = Unhealthy
 ↓
ALB stops sending new traffic to that target.


4. LOCAL VS NETWORK ACCESS

Important troubleshooting lesson:

localhost:5000/health → 200 OK

only proves that Flask responds locally.

It does NOT prove:

- ALB can reach Flask
- Network routing works
- Firewall allows traffic
- Security Group allows traffic
- Correct target port is configured
- Health-check configuration is correct

Mental model:

Local test:
"Does the application respond locally?"

Network test:
"Can the ALB reach the application?"


5. HEALTH-CHECK CONFIGURATION

Example failure:

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
ALB marks target unhealthy.

Fix:

Change the ALB health-check path to:

/health

Important lesson:

The ALB does not automatically know which endpoint should be used to test the application.


6. ALB ERROR TROUBLESHOOTING

502 Bad Gateway:

Potentially indicates that the ALB had difficulty getting a valid response from a backend target.

Investigate:

- Flask application
- Target port
- Listening interface
- Security Groups
- Network connectivity
- Application response


503 Service Unavailable:

Can indicate that the ALB has no healthy targets available.

Investigate:

- Target Group
- Health-check configuration
- Flask application
- Port
- Listening interface
- Security Groups
- Routing
- Network ACLs
- Firewall


7. TROUBLESHOOTING PATH

Use the actual traffic path:

DNS
 ↓
ALB
 ↓
Listener
 ↓
Target Group
 ↓
Health Check
 ↓
Network
 ↓
Flask
 ↓
Database

Important troubleshooting principle:

Once a layer has been proven healthy, move forward instead of repeatedly checking it.


8. AUTO SCALING

Auto Scaling adjusts application capacity according to demand.

Example:

Traffic increases
 ↓
CPU increases
 ↓
CloudWatch metric
 ↓
Scaling policy
 ↓
Auto Scaling Group
 ↓
New EC2 instance
 ↓
Health check
 ↓
Target Group
 ↓
ALB
 ↓
Traffic distributed


9. CLOUDWATCH

CloudWatch collects and monitors metrics and logs.

In the Auto Scaling architecture:

EC2
 ↓
CPU utilization
 ↓
CloudWatch
 ↓
Scaling policy evaluates metric
 ↓
Auto Scaling Group
 ↓
Capacity adjustment

Key mental model:

CloudWatch observes.

Auto Scaling acts.


10. SCALE OUT VS SCALE IN

Scale OUT:

2 → 3 → 4

Add instances when demand increases.

Scale IN:

4 → 3 → 2

Remove unnecessary instances when demand decreases.

Capacity boundaries:

Minimum:
→ Lowest number of instances allowed.

Desired:
→ Current target capacity.

Maximum:
→ Highest number of instances allowed.


11. TARGET TRACKING

Target Tracking uses a target value.

Example:

Target CPU = 60%

If CPU becomes high:

CPU → 85%
 ↓
Scale OUT
 ↓
More instances
 ↓
CPU moves toward target.

If CPU becomes very low:

CPU → 20%
 ↓
Scale IN
 ↓
Fewer instances.

Important:

The target value is the desired utilization.

It is NOT the minimum number of instances.


12. STEP SCALING

Step Scaling uses different actions at different thresholds.

Example:

CPU < 30%
→ Remove 1 instance

CPU 30–70%
→ No change

CPU 70–85%
→ Add 1 instance

CPU > 85%
→ Add 2 instances

Key distinction:

Target Tracking:
→ "Try to keep utilization around this target."

Step Scaling:
→ "Take this specific action when this threshold is reached."


13. PROACTIVE VS REACTIVE SCALING

Reactive scaling:

Traffic spike
 ↓
Metric increases
 ↓
Scaling policy
 ↓
New instances

Problem:

New instances take time to launch, configure, and pass health checks.

Proactive scaling:

Known traffic spike
 ↓
Scale BEFORE demand
 ↓
Capacity already available

Example:

08:55
2 → 6 instances

09:00
Traffic spike

6 instances are already available.

Key principle:

Proactive scaling:
→ Good for predictable demand.

Reactive scaling:
→ Good for unexpected demand.

Production systems can combine both approaches.


14. LAUNCH TEMPLATES

Launch Template:

→ Blueprint for creating EC2 instances.

Can define:

- AMI
- Instance type
- Security Group
- IAM role
- User Data
- Other instance configuration

Mental model:

Launch Template = Blueprint


15. LAUNCH TEMPLATE + AUTO SCALING

Relationship:

Launch Template
 ↓
Blueprint
 ↓
Auto Scaling Group
 ↓
Launches EC2 instances

If an unhealthy instance needs replacement:

Bad EC2
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
ALB


16. ALB + AUTO SCALING RESPONSIBILITIES

ALB:

- Receives traffic
- Performs health checks
- Selects healthy targets
- Distributes traffic

Auto Scaling Group:

- Maintains desired capacity
- Adds instances
- Removes instances
- Replaces unhealthy instances when configured to do so

Launch Template:

- Defines how new instances should be configured

CloudWatch:

- Collects and monitors metrics/logs


17. FULL ARCHITECTURE

Reconstructed the architecture from memory:

Internet
 ↓
DNS
 ↓
ALB
 ↓
Target Group
 ↓
EC2 instances
 ↓
Application
 ↓
Database

With Auto Scaling:

                 ALB
                  ↓
            Target Group
             ↙        ↘
           EC2-A     EC2-B
             ↑          ↑
             └── ASG ───┘
                  ↑
             CloudWatch

With high availability:

AZ-A                    AZ-B
EC2-A                   EC2-B
  │                        │
  └──────── ALB ───────────┘

Database:

Multi-AZ Database
→ Availability / failover


18. COMPLETE SCALING FLOW

The complete mental model developed today:

CloudWatch
 ↓
Metric
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

Example:

CPU = 85%
 ↓
CloudWatch observes high CPU
 ↓
Scaling policy triggers
 ↓
Auto Scaling Group scales OUT
 ↓
Launch Template provides instance configuration
 ↓
New EC2 launches
 ↓
Application starts
 ↓
Health check passes
 ↓
Target becomes healthy
 ↓
ALB sends traffic to it


19. IMPORTANT ARCHITECTURAL DISTINCTION

Do not think of Auto Scaling as being directly in the traffic path.

Incorrect mental model:

ALB
 ↓
Auto Scaling
 ↓
EC2

Better model:

                 ALB
                  ↓
            Target Group
             ↙        ↘
           EC2-A     EC2-B
             ↑          ↑
             └── ASG ───┘
                  ↑
             CloudWatch

The ASG manages the instances.

The ALB distributes traffic to the instances.


20. TODAY'S TROUBLESHOOTING LESSON

A major lesson today was:

"What exactly has my test proven?"

Example:

localhost:5000/health → 200 OK

This proves:

→ Flask responds locally.

It does NOT prove:

→ ALB can reach Flask
→ Network routing works
→ Firewall permits traffic
→ Security Group permits traffic
→ Health-check path is correct
→ Target port is correct


21. TODAY'S PERFORMANCE

ALB:
GOOD

Health Checks:
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

Multi-AZ Architecture:
GOOD

Troubleshooting:
IMPROVING STRONGLY

DNS:
NEEDS CONTINUED REINFORCEMENT


22. KEY CONCEPTS TO REMEMBER

ALB:
→ Where should traffic go?

Auto Scaling:
→ How many instances should exist?

Launch Template:
→ How should a new instance be configured?

CloudWatch:
→ What is happening?

Health Check:
→ Is this target healthy?

Target Group:
→ Which instances are available to the ALB?

Multi-AZ:
→ How do we maintain availability if an AZ fails?

Read Replica:
→ How do we scale database reads?

Cache:
→ How do we reduce repeated database requests?


23. TOMORROW'S PLAN

Start with a short retrieval exercise covering:

- ALB
- Target Groups
- Health Checks
- Auto Scaling
- CloudWatch
- Launch Templates
- Multi-AZ

Then move toward practical implementation:

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
Flask application

After understanding the architecture, begin translating it into Terraform so the project becomes reproducible.


FINAL NOTE

Today's session was productive.

The biggest improvement was moving from understanding individual AWS services to understanding how they cooperate as one system.

Core architecture:

CloudWatch observes
 ↓
Auto Scaling manages capacity
 ↓
Launch Template defines instances
 ↓
Health Checks validate instances
 ↓
Target Group organizes targets
 ↓
ALB distributes traffic

Key principle:

ALB protects traffic from unhealthy instances.

Auto Scaling maintains the required capacity.

Launch Templates provide the blueprint.

CloudWatch provides the metrics.

Multi-AZ provides high availability.

Good progress today.