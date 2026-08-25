DAILY LOG — CLOUD / AWS MASTERY — DAY 1

Date: 25 August 2026

Today's focus:
☁️ AWS Cloud Fundamentals
🌐 VPC Networking
🔐 Security Groups
🛡️ IAM Fundamentals
💰 AWS Cost Awareness


1. CLOUD FUNDAMENTALS

Connected the existing CI/CD pipeline to the Cloud concept.

Existing pipeline:

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
Deploy

New question:

"Where does the Docker container actually run?"

Learned that Cloud provides abstracted infrastructure such as:

- Compute
- Networking
- Storage
- Databases
- Identity / Security

The underlying physical infrastructure still exists; the Cloud provider manages and abstracts much of it.


2. CLOUD SERVICE MODELS

Reviewed:

IaaS:
Infrastructure as a Service

PaaS:
Platform as a Service

SaaS:
Software as a Service

Mental model:

IaaS → "Give me infrastructure."
PaaS → "Give me a platform."
SaaS → "Give me the finished software."


3. AWS REGIONS

Learned that an AWS Region is a geographic infrastructure location.

Reasons for choosing a region include:

- Latency
- Compliance / data residency
- Availability
- Cost

Region:

Region
 ├── Availability Zone
 ├── Availability Zone
 └── Availability Zone


4. AVAILABILITY ZONES

An Availability Zone is an isolated infrastructure location within a Region.

Key concept:

AZ = failure domain

Using multiple AZs improves availability because a failure affecting one AZ does not necessarily take down the entire application.

Example:

AZ-A ❌
AZ-B ✅

A properly designed application can continue operating from AZ-B.


5. HIGH AVAILABILITY

Compared:

Two containers in the same AZ:

AZ-A
 ├── Container 1
 └── Container 2

If AZ-A fails:

Both may become unavailable.

Better:

AZ-A
 └── App-A

AZ-B
 └── App-B

If AZ-A fails:

App-A ❌
App-B ✅


6. LOAD BALANCING

Learned that a load balancer can distribute traffic across healthy application instances.

Concept:

Internet
   ↓
Load Balancer
   ├── AZ-A → App-A
   └── AZ-B → App-B

If App-A becomes unhealthy, the load balancer can stop sending traffic to it.

Important distinction:

Load Balancer:
→ distributes traffic

Auto Scaling:
→ changes the number of application instances based on demand

A load balancer does not automatically create additional capacity by itself.


7. DATABASE HIGH AVAILABILITY

Learned that database availability depends on the database service and configuration.

Do NOT assume:

Database
 ↓
Automatically replicated everywhere

Instead, configurations can provide patterns such as:

AZ-A
 └── Primary DB
       │
       │ replication
       ▼
AZ-B
 └── Standby DB

Multi-AZ and multi-region replication are different concepts.

AZ failure:
→ Multi-AZ architecture

Regional failure:
→ Multi-region architecture / disaster recovery strategy

Important principle:

More redundancy ≠ automatically better.

Replication introduces:

- Cost
- Complexity
- Consistency considerations
- Replication lag
- Operational overhead


8. VPC FUNDAMENTALS

Introduced AWS VPC.

Example:

VPC:
10.0.0.0/16

The VPC provides the network boundary for AWS resources.

Within it we can create:

- Subnets
- Route tables
- Security controls
- Internet connectivity


9. SUBNET DESIGN

Corrected the idea that every component necessarily needs its own subnet.

Subnets should be designed around:

- Network boundaries
- Routing requirements
- Exposure
- Availability Zones
- Security architecture

Example:

VPC
│
├── AZ-A
│    ├── Public subnet
│    └── Private subnet
│
└── AZ-B
     ├── Public subnet
     └── Private subnet

Important:

A subnet is a network boundary, not simply a container for one individual service.


10. PUBLIC VS PRIVATE RESOURCES

Learned the principle:

Only expose resources that actually need public access.

Typical architecture:

Internet
   ↓
Public Load Balancer
   ↓
Private Application
   ↓
Private Database

The application generally does not need a public IP.

The database should generally not be directly Internet-accessible.


11. ROUTING

Reconnected AWS networking to previously learned networking fundamentals.

Routing answers:

"Where should this packet go?"

AWS implements routing using Route Tables.

Example:

Destination        Target

10.0.0.0/16        local
0.0.0.0/0          Internet Gateway

Important distinction:

Route Table:
→ determines where traffic goes

Security Group:
→ determines whether traffic is allowed


12. INTERNET GATEWAY

Learned that an Internet Gateway provides the VPC's connection to the Internet.

Conceptually:

Public subnet
   ↓
Internet Gateway
   ↓
Internet

Important correction:

An Internet Gateway is not itself a normal application firewall.

Traffic filtering is handled through mechanisms such as:

- Security Groups
- Network ACLs


13. NAT GATEWAY

Initially confused NAT Gateway with Internet Gateway.

Corrected mental model:

Private application
       ↓
Private subnet
       ↓
NAT Gateway
       ↓
Internet Gateway
       ↓
Internet

Purpose:

Allow private resources to initiate outbound Internet connections while preventing direct unsolicited inbound connections through that NAT path.

Important:

NAT Gateway is generally placed in a public subnet so that it can reach the Internet through the Internet Gateway.


14. INTERNET GATEWAY VS NAT GATEWAY

Internet Gateway:

→ Provides VPC connectivity to the Internet.

NAT Gateway:

→ Provides outbound Internet access for private resources while keeping those resources from being directly Internet-accessible through the NAT path.

Route Table:

→ Determines where traffic goes.


15. SECURITY GROUPS

Introduced Security Groups as AWS virtual traffic-control mechanisms associated with resources such as EC2 instances.

Security Group mental model:

Resource
   ↓
Security Group
   ↓
Allow / restrict traffic

Example architecture:

Internet
   ↓ TCP 443
Load Balancer
   ↓ TCP 5000
Application
   ↓ TCP 5432
Database


16. LEAST-PRIVILEGE SECURITY GROUP DESIGN

Designed the following conceptual rules:

LB-SG:

TCP 443
Source: Internet

APP-SG:

TCP 5000
Source: LB-SG

DB-SG:

TCP 5432
Source: APP-SG

Result:

Internet → Load Balancer       ✅
Internet → Application          ❌
Internet → Database             ❌

Load Balancer → Application     ✅
Application → Database          ✅

Important principle:

Prefer the smallest necessary trust boundary rather than opening ports to 0.0.0.0/0 unnecessarily.


17. SECURITY GROUPS ARE STATEFUL

Learned that Security Groups are stateful.

If an allowed outbound connection is established, the corresponding response traffic is automatically allowed as part of that established connection.

This differs conceptually from stateless network filtering.


18. ROUTING VS SECURITY GROUP

Important distinction:

Route Table:

"Where should traffic go?"

Security Group:

"Is this traffic allowed?"

Both need to be correct.

Example:

Packet
 ↓
Route table
 ↓
Where?
 ↓
Security Group
 ↓
Allowed?
 ↓
Destination


19. NETWORK TROUBLESHOOTING

Scenario:

Private application needs:

curl https://pypi.org

but receives a timeout.

Potential investigation:

DNS
 ↓
Route table
 ↓
NAT Gateway
 ↓
Public-subnet route
 ↓
Internet Gateway
 ↓
Security controls
 ↓
Internet

Important troubleshooting principle:

Do not immediately change configuration.

Verify each layer using evidence.


20. IAM FUNDAMENTALS

Introduced AWS IAM.

Core model:

WHO
 ↓
CAN DO WHAT
 ↓
TO WHICH RESOURCE

IAM handles identity and authorization.

Important distinction:

Authentication:
→ Who are you?

Authorization:
→ What are you allowed to do?


21. IAM LEAST PRIVILEGE

Scenario:

EC2 application needs to read objects from an S3 bucket.

Correct answer:

Give the application an identity with only the required read permissions.

Avoid:

- Hardcoded access keys
- Full S3 permissions
- Administrator permissions

Mental model:

EC2
 ↓
IAM Role
 ↓
Permissions Policy
 ↓
Specific S3 access


22. IAM ROLES

Learned that IAM Roles provide identities that can be assumed by trusted principals.

Example:

EC2
 ↓
IAM Role
 ↓
Temporary credentials
 ↓
AWS API


23. TRUST POLICY VS PERMISSIONS POLICY

This was identified as an important concept to reinforce next lesson.

Trust Policy:

"WHO can assume this role?"

Permissions Policy:

"WHAT can this role do?"

Mental model:

              IAM ROLE
                 │
        ┌────────┴────────┐
        ↓                 ↓
 TRUST POLICY      PERMISSIONS POLICY
        │                 │
 "WHO can assume?"  "WHAT can they do?"


24. IAM TROUBLESHOOTING

Scenario:

EC2 has an S3 GetObject permission but receives AccessDenied.

Correct investigation sequence:

EC2
 ↓
Is intended IAM Role attached?
 ↓
Can EC2 assume the role?
 ↓
Are temporary credentials available?
 ↓
Does permissions policy allow s3:GetObject?
 ↓
Does permission apply to the specific resource?
 ↓
Is there an explicit Deny?
 ↓
Are there bucket/resource policy restrictions?


25. AWS COST AWARENESS

Discussed practicing AWS without unnecessarily spending large amounts of money.

Important:

Not every AWS service/resource should be assumed to be free.

Especially important to watch:

NAT Gateway
RDS
Load Balancers
Compute resources
Data transfer
Other metered services

Practice strategy:

Create
 ↓
Practice
 ↓
Test
 ↓
Destroy


26. CLOUD CAPSTONE PLAN

At the end of the Cloud unit, the goal is to build a production-style AWS deployment rather than a tutorial-following exercise.

Expected architecture:

Internet
   ↓
DNS
   ↓
Load Balancer
   ↓
Private Applications
   ↓
Private Database

Across multiple Availability Zones.

The project will include:

- VPC
- CIDR design
- Public/private subnets
- Route tables
- Internet Gateway
- NAT where appropriate
- Security Groups
- IAM
- Container deployment
- Database
- High availability
- Monitoring
- CI/CD
- Failure testing
- Troubleshooting
- Cost awareness


27. CAPSTONE MASTERY STANDARD

The Cloud project will not be considered complete simply because it deploys.

Required progression:

Understand
 ↓
Design
 ↓
Implement
 ↓
Troubleshoot
 ↓
Break deliberately
 ↓
Recover
 ↓
Secure
 ↓
Explain tradeoffs
 ↓
Apply to unfamiliar scenarios
 ↓
MASTERED


28. CURRENT CLOUD STATUS

Cloud fundamentals              🟢
Regions / AZs                   🟢
High availability               🟢
VPC fundamentals                🟢
Subnets                         🟢
Routing                         🟢
Internet Gateway                🟢
NAT Gateway                     🟢
Security Groups                 🟢
Least privilege                 🟢
IAM fundamentals                🟢
IAM Roles                       🟡
Trust vs Permissions policies   🟡 → revisit next lesson


29. NEXT LESSON

Begin with an IAM Role retrieval exercise.

Specifically revisit:

IAM Role
 ↓
Trust Policy
 ↓
Who can assume it?

and:

IAM Role
 ↓
Permissions Policy
 ↓
What can it do?

Then continue with:

IAM Users
IAM Policies
IAM authorization evaluation
EC2
Storage
Databases


FINAL REFLECTION

Today's biggest achievement was connecting previously learned networking concepts to AWS infrastructure.

The mental model is becoming:

Networking fundamentals
        ↓
AWS implementation
        ↓
VPC
        ↓
Subnets
        ↓
Routes
        ↓
Internet/NAT
        ↓
Security Groups
        ↓
IAM
        ↓
Cloud architecture

The goal remains mastery rather than simply completing AWS services quickly.