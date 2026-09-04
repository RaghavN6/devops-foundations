DAILY LOG — AWS / DEVOPS

Date: 2026-09-04

TODAY'S FOCUS:
Building the foundational understanding of WHY DevOps, Cloud, and modern infrastructure technologies exist, rather than memorizing tools and services.

1. DEVOPS — WHY IT EXISTS
- Before DevOps, Development and Operations often worked with different goals:
  - Developers focused on features and delivery.
  - Operations focused on stability, availability, security, and reliability.
- Agile made software development faster, but infrastructure and operations could remain slow.
- DevOps evolved to reduce the friction between development and operations.
- Core DevOps idea:
  Build → Test → Deploy → Operate → Observe → Improve
- DevOps is NOT simply a collection of tools.
- DevOps is a combination of culture, processes, automation, measurement, and engineering practices for reliably delivering and operating software.

2. PHYSICAL SERVER ERA
- Traditionally, companies bought and operated physical servers.
- They were responsible for:
  - Hardware
  - Data centers
  - Networking
  - Power
  - Cooling
  - Storage
  - Operating systems
  - Applications
  - Maintenance
- Major problems:
  - Expensive infrastructure
  - Slow provisioning
  - Hardware failures
  - Difficult capacity planning
  - Poor resource utilization
  - Need to provision for peak traffic

3. REDUNDANCY AND HIGH AVAILABILITY
- A single physical server creates a single point of failure.
- Companies introduced multiple servers and load balancing.
- Core principle learned:
  Redundancy allows workloads to survive individual infrastructure failures.
- Example:
  Internet → Load Balancer → Server A / Server B
- If Server A fails, traffic can continue through Server B.

4. CAPACITY PROBLEM
- Companies often had to buy enough hardware for peak demand.
- Example:
  Normal traffic = 100 requests/sec
  Peak traffic = 5,000 requests/sec
- Buying enough infrastructure for 5,000 requests/sec means much of the capacity may sit unused during normal periods.
- This created a major economic and operational problem.

5. VIRTUALIZATION
- Virtualization allowed multiple isolated virtual machines to share one physical server.
- Hypervisor sits between physical hardware and VMs.
- Example:
  Physical Server
      ↓
  Hypervisor
      ↓
  VM1 / VM2 / VM3
- Benefits:
  - Better hardware utilization
  - Isolation between workloads
  - Faster provisioning
  - More flexible resource allocation
- Important distinction:
  Virtualization does not eliminate the physical infrastructure problem; it makes the physical infrastructure more efficiently utilized.

6. WHY CLOUD EMERGED
- Cloud providers operate massive data centers and infrastructure on behalf of customers.
- Companies can consume infrastructure instead of owning and operating all of it themselves.
- Instead of:
  Buy hardware → install → configure → operate
- Cloud allows:
  API/Console → Request resource → Consume resource
- Cloud changed infrastructure economics and provisioning speed.
- Core mental model:
  "How many servers should we buy?"
  became:
  "How much infrastructure do we need right now, and how can we provision it programmatically?"

7. CLOUD + AUTOMATION
- Cloud provides infrastructure and managed services.
- Automation can dynamically adjust infrastructure.
- Example:
  Traffic increases
      ↓
  Monitoring detects change
      ↓
  Scaling policy
      ↓
  Additional compute
- Important:
  Cloud ≠ Auto Scaling.
- Auto Scaling is one capability available within cloud platforms.

8. IAAS — INFRASTRUCTURE AS A SERVICE
- EC2 is an example of IaaS.
- AWS manages:
  - Physical servers
  - Data centers
  - Power
  - Cooling
  - Physical networking
  - Hypervisor
- Customer generally manages:
  - Operating system
  - OS patches
  - Runtime
  - Dependencies
  - Application
  - Application configuration
- Important principle:
  Renting infrastructure does not mean AWS manages everything inside the VM.

9. SHARED RESPONSIBILITY
- Cloud security and operations responsibilities are divided between provider and customer.
- With EC2:
  AWS manages the underlying physical infrastructure.
  Customer manages the guest operating system and workload.
- Key lesson:
  Always ask:
  "Which layer am I responsible for, and which layer does the provider manage?"

10. ABSTRACTION — IAAS → MANAGED SERVICES → SAAS
- Cloud platforms provide progressively higher levels of abstraction.
- More abstraction generally means:
  - Less infrastructure to manage
  - Less operational burden
  - Potentially less control
- Core trade-off:
  Control ↔ Operational effort
- IaaS:
  More control, more responsibility.
- Managed services:
  Provider operates more of the underlying infrastructure.
- SaaS:
  Provider operates essentially the entire software platform.

11. TOOL/TECHNOLOGY EVOLUTION
The historical chain we established:

Physical Servers
    ↓
Problem: expensive, slow, poor utilization
    ↓
Virtualization
    ↓
Problem: infrastructure still expensive to operate
    ↓
Cloud Computing
    ↓
Problem: manually managing cloud infrastructure doesn't scale
    ↓
Infrastructure as Code
    ↓
Problem: application deployment remains complicated
    ↓
Containers
    ↓
Problem: managing many containers becomes difficult
    ↓
Orchestration
    ↓
Kubernetes / managed container platforms

12. CORE MASTERY INSIGHT
- Do not learn:
  "EC2 = AWS virtual machine"
  and stop there.
- Instead learn:
  - What problem did VMs solve?
  - Why did cloud providers emerge?
  - Why do we need load balancing?
  - Why do we need autoscaling?
  - Why do we need containers?
  - Why do we need orchestration?
  - Why do we need IaC?
  - Why do we need observability?
- The goal is to understand the engineering problem first and the tool second.

13. FUTURE TOOL/CAREER DIRECTION
The broader professional ecosystem we identified includes:

Source Control:
- Git
- GitHub
- GitLab
- Bitbucket

CI/CD:
- GitHub Actions
- GitLab CI
- Jenkins
- Argo CD
- Azure DevOps

Containers:
- Docker
- Podman
- containerd
- Buildah

Orchestration:
- Kubernetes
- ECS
- OpenShift
- Nomad

Infrastructure as Code:
- Terraform
- OpenTofu
- CloudFormation
- Pulumi

Configuration Management:
- Ansible
- Puppet
- Chef
- Salt

Observability:
- Prometheus
- Grafana
- OpenTelemetry
- CloudWatch
- Datadog
- Elastic
- Splunk

Networking/Troubleshooting:
- curl
- dig
- ss
- tcpdump
- traceroute
- Wireshark
- netcat

Security:
- IAM
- Vault
- Secrets Manager
- SOPS
- Trivy
- OPA
- Kyverno
- Falco

Databases:
- PostgreSQL
- MySQL
- Redis
- DynamoDB
- MongoDB

Messaging/Distributed Systems:
- Kafka
- RabbitMQ
- SQS
- SNS
- NATS
- Kinesis

14. PROFESSIONAL-LEVEL THINKING
- Senior engineers are not valuable simply because they know many tools.
- They understand:
  - Linux
  - Networking
  - Programming
  - Cloud architecture
  - Distributed systems
  - Security
  - Automation
  - Observability
  - Reliability
  - Scalability
- Tools are implementations of these underlying concepts.
- A senior engineer should also understand when NOT to use a particular technology.

15. MASTERY LADDER
For each concept/tool, future study should progress through:

Level 1 — Vocabulary
- What is it?

Level 2 — Purpose
- What problem does it solve?

Level 3 — Architecture
- How does it work?

Level 4 — Usage
- Can I build with it?

Level 5 — Troubleshooting
- Can I diagnose failures?

Level 6 — Internals
- What happens underneath?

Level 7 — Trade-offs
- When should I NOT use it?

Level 8 — Architecture
- Can I design a production system with it?

Level 9 — Scale
- Can I operate it under serious load?

Level 10 — Engineering judgment
- Can I compare alternatives and defend a technical decision?

PERFORMANCE TODAY:
- Strong conceptual understanding of why virtualization and cloud emerged.
- Correctly identified that companies use cloud to avoid owning and maintaining expensive physical infrastructure.
- Correctly connected cloud elasticity/Auto Scaling with changing traffic.
- Correctly understood that EC2 customers are generally responsible for their guest OS and dependencies.
- Most important improvement identified:
  Move away from memorizing individual services and toward understanding the underlying problem each technology solves.

AREAS TO REINFORCE:
- Difference between virtualization and cloud.
- Difference between cloud and Auto Scaling.
- IaaS vs managed services vs SaaS.
- Shared Responsibility Model.
- The historical progression from physical infrastructure to cloud-native systems.
- Why companies choose one tool over another.
- The operational and economic trade-offs behind technology choices.

NEXT SESSION:
Continue from:
"Why should a company use RDS instead of installing PostgreSQL on EC2?"

Then expand into:
- Managed services
- PaaS
- SaaS
- AWS service evolution
- Why companies choose managed vs self-managed infrastructure
- How professional companies make technology decisions

After the historical foundation is complete, build a personalized:
DEVOPS / CLOUD MASTERY MATRIX

For every major technology:
- Purpose
- Problem solved
- Alternatives
- Why professionals choose it
- Trade-offs
- Required knowledge depth
- Hands-on projects
- Production troubleshooting skills
- Senior-level expectations
- Expert-level expectations
- Recommended resources

FINAL TAKEAWAY:
The goal is NOT to become someone who memorizes AWS, Docker, Kubernetes, Terraform, Jenkins, and hundreds of commands.

The goal is to become an engineer who can look at a business or technical problem and reason:

"What problem are we solving?"
"What constraints do we have?"
"What technology solves it?"
"What are the alternatives?"
"What are the trade-offs?"
"How will it behave under failure?"
"How will we monitor it?"
"How will we automate it?"
"How will we scale it?"

Once that foundation is strong, individual tools become much easier to learn and much harder to forget.