DAILY LOG — FOUNDATION STRESS TEST → CLOUD PREPARATION

Today was a foundation reinforcement and roadmap-planning session.

The main objective was to determine whether my existing DevOps foundations were strong enough to begin the Cloud/AWS module.

Rather than simply rereading everything, I tested my understanding through troubleshooting scenarios and practical questions.


1. FOUNDATION STRESS TEST

Areas tested:

- Git
- Linux
- Networking
- CIDR / subnetting
- DNS
- TCP / UDP
- Troubleshooting
- CI/CD concepts


2. GIT

Reinforced the Git mental model:

Working Tree
 ↓
Staging Area
 ↓
Commit
 ↓
Local Repository
 ↓
Remote Repository

Important distinctions:

- Working tree = actual files currently being worked on.
- Staging area = exact changes selected for the next commit.
- Commit = snapshot stored in Git history.
- Local repository = local Git history.
- Remote repository = repository hosted remotely, such as GitHub.

Important terminology:

origin = remote name
main = branch name


3. GIT HISTORY / SYNCHRONIZATION

Encountered a non-fast-forward push scenario.

Example:

REMOTE:
A → B → C

LOCAL:
A → B → D

The branches have diverged.

The remote contains commits that the local branch does not have.

Important workflow:

git fetch origin
 ↓
Inspect remote history
 ↓
Merge or rebase
 ↓
Resolve conflicts if required
 ↓
Test
 ↓
git push origin main

Important concept:

git fetch does not automatically integrate the remote changes into the current branch.

Force pushing can overwrite remote branch history and potentially remove other people's commits.

This reinforced that Git is primarily about managing version history and distributed changes, not simply storing files.


4. LINUX PROCESSES AND PORTS

Troubleshooting scenario:

Flask should be running on port 5000 but:

curl http://localhost:5000/health

returns:

Connection refused

Reasoning:

Check whether the process exists.

Possible commands:

ps aux | grep python
pgrep -af python

Then check listening sockets:

ss -ltnp
ss -ltnp | grep :5000

Also investigate:

- Listening port
- Listening interface
- Process ownership
- Application logs


5. LISTENING INTERFACES

Important concept learned:

127.0.0.1:5000

means the application is listening only on the local loopback interface.

Whereas:

0.0.0.0:5000

means it is listening on all IPv4 interfaces.

This distinction will become important for:

- Docker
- AWS
- Kubernetes
- Load balancers
- Production deployments


6. LINUX FILE PERMISSIONS

Successfully analyzed:

-rw-r--r--

Meaning:

Owner:
read + write

Group:
read

Others:
read

No execute permission exists.

Therefore:

./deploy.sh

fails with:

Permission denied

Correct fix:

chmod +x deploy.sh

Important permission model:

r = read
w = write
x = execute

Also learned that `x` has a different practical meaning for directories: it allows traversal/access through the directory.


7. NETWORKING TROUBLESHOOTING

Scenario:

Flask is running and listening on:

0.0.0.0:5000

but another machine receives:

Connection timed out

Reasoning:

Client
 ↓
Routing / network
 ↓
Firewall
 ↓
TCP connection
 ↓
Port
 ↓
Application

Useful diagnostic tools:

ping
traceroute
nc

Important distinction:

ping tests ICMP reachability.

It does NOT prove that TCP port 5000 is reachable.

Similarly, traceroute helps investigate the network path but does not prove that a specific TCP port is accessible.

For port-specific testing:

nc -vz <host> <port>


8. CONNECTION REFUSED VS TIMEOUT

Reinforced an important troubleshooting clue.

Connection refused often indicates:

- Host is reachable.
- Nothing is accepting the connection on that port.
- Or something actively rejected the connection.

Connection timed out often suggests traffic may be getting dropped somewhere, such as:

- Firewall
- Routing issue
- Security control
- Network path


9. CIDR / SUBNETTING

Initially discovered that CIDR/subnetting knowledge had become weak.

Rebuilt the concept from fundamentals.

IPv4:

32 bits

For a subnet:

host bits = 32 - prefix

total addresses = 2^(host bits)

Traditional usable IPv4 addresses:

total - 2

Example:

/24

32 - 24 = 8 host bits

2^8 = 256 total

256 - 2 = 254 traditionally usable


10. CIDR PRACTICE

Successfully solved:

/25:

32 - 25 = 7

2^7 = 128 total

126 traditionally usable


/28:

32 - 28 = 4

2^4 = 16 total

14 traditionally usable


A /24 split into four equal subnets:

10.0.0.0/26
10.0.0.64/26
10.0.0.128/26
10.0.0.192/26

Each contains:

64 total addresses
62 traditionally usable addresses


11. CIDR IMPORTANT AWS CAVEAT

The traditional IPv4 calculation:

total - 2

is a networking fundamentals rule.

AWS reserves additional addresses inside each subnet.

Therefore AWS subnet usable-address calculations differ from the traditional model.

This will be covered properly during the VPC module.


12. DNS

Established the core mental model:

Domain name
 ↓
DNS
 ↓
IP address
 ↓
Network connection
 ↓
Application request

DNS is effectively a distributed directory that maps names to destinations.

Example:

api.example.com
 ↓
DNS
 ↓
IP address

DNS does not deliver the webpage/application response itself.

The client uses the resolved address to establish the connection.


13. DNS TROUBLESHOOTING

Scenario:

curl http://10.0.0.20:5000/health

works.

But:

curl http://api.example.com/health

fails with:

Could not resolve host

Reasoning:

Application works.
IP connectivity works.
Port works.

Therefore investigate DNS rather than immediately modifying the application.

Useful tools:

dig
nslookup

Important troubleshooting principle:

Do not investigate components that the evidence has already cleared.


14. TCP VS UDP

Initially remembered the general differences but incorrectly associated TCP's handshake with security.

Corrected understanding:

TCP provides:

- Connection-oriented communication
- Reliable delivery
- Ordered delivery
- Retransmission
- Flow control
- Congestion control

TCP does NOT inherently provide encryption.

TLS provides encryption/authentication for HTTPS.

Conceptually:

HTTPS
 ↓
HTTP
 ↓
TLS
 ↓
TCP
 ↓
IP


UDP:

- Connectionless
- No built-in delivery guarantee
- No built-in ordering guarantee
- Lower protocol overhead

UDP can be useful for:

- Gaming
- Real-time communication
- Streaming
- DNS
- Other latency-sensitive applications

Important distinction:

TCP ≠ secure
UDP ≠ insecure

TCP and UDP are transport protocols; security/encryption is a separate concern.


15. LAYERED NETWORK TROUBLESHOOTING

Established the overall troubleshooting chain:

DNS
 ↓
Network / routing
 ↓
TCP
 ↓
Port / listening socket
 ↓
TLS / HTTP
 ↓
Application

Example diagnostic tools:

DNS:
dig / nslookup

Network:
ping / traceroute

TCP/port:
nc

Local listening sockets:
ss

Application:
logs / process inspection / health endpoint


16. FOUNDATION STRESS TEST RESULT

Git:
🟢

Linux:
🟢

Permissions:
🟢

Processes / ports:
🟢

CIDR:
🟢 after reinforcement

DNS:
🟢

TCP / UDP:
🟢 after conceptual correction

Network troubleshooting:
🟢

Layered debugging:
🟢


17. IMPORTANT LEARNING DISCOVERY

I initially felt that many foundations had been forgotten.

However, the stress test showed that the underlying understanding is still present in many areas.

The main issue is:

RECOGNITION > RECALL

Knowledge sometimes needs to be reactivated before it becomes available from memory.

Example:

"I forgot CIDR."

 ↓

Rebuilt the underlying formula.

 ↓

Successfully solved:

/25
/28
/24 → four /26 subnets

Therefore the solution is not to repeatedly restart the entire foundation module.

Instead use:

Spaced recall
+
Practical application
+
Troubleshooting scenarios


18. NEW FOUNDATION REINFORCEMENT STRATEGY

Going forward:

Learn new concept
 ↓
Apply it
 ↓
Encounter old foundation
 ↓
Quickly revise it
 ↓
Apply it again
 ↓
Recall later
 ↓
Strengthen long-term memory

Older foundations will be periodically tested without preparation.


19. ROADMAP STRATEGY

The primary goal is:

🏆 MASTERY

Not:

- Finishing quickly
- Collecting certificates
- Memorizing services
- Completing tutorials

Mastery standard:

Understand
 ↓
Explain
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
Understand tradeoffs
 ↓
Apply to unfamiliar scenarios
 ↓
Teach / explain
 ↓
MASTERED


20. SEMESTER DEADLINE

Target:

Mid-to-late February 2027

Approximately six months available.

The semester-end goal is:

Strong junior-level DevOps/Cloud capability
+
2–3 serious portfolio projects
+
Internship readiness


21. INTERNSHIP OBJECTIVE

Long-term sequence:

Learn
 ↓
Build
 ↓
Develop strong portfolio
 ↓
Become internship-ready
 ↓
Apply for paid remote internships
 ↓
Gain real professional experience
 ↓
Build funds
 ↓
Pursue certifications later
 ↓
Continue advanced mastery


22. TIME COMMITMENT

Recommended normal target:

2.5–3 focused hours/day

Approximately:

18–22 focused hours/week

The priority is not the number of hours.

The priority is:

Learn
 ↓
Build
 ↓
Break
 ↓
Troubleshoot
 ↓
Document
 ↓
Recall


23. IMPORTANT PRIORITY CHANGE

The internship is NOT the primary objective.

The hierarchy is:

1. 🏆 Mastery
2. 🛠️ Practical ability
3. 🧠 Strong fundamentals
4. 💼 Internship readiness
5. 📁 Portfolio
6. 🎓 Certifications

The February deadline is a milestone.

Mastery remains the ultimate goal.


24. CLOUD MODULE — NEXT

The foundation stress test is now complete enough to begin Cloud.

Next module:

☁️ CLOUD / AWS

Starting from:

What is Cloud?
 ↓
Regions
 ↓
Availability Zones
 ↓
VPC
 ↓
Subnets
 ↓
Routing
 ↓
IAM
 ↓
Compute
 ↓
Where does the Docker container actually run?


25. NEXT SESSION OBJECTIVE

Begin AWS/Cloud Mastery from first principles.

The goal is NOT to memorize AWS services.

Instead:

Understand the problem
 ↓
Design the architecture
 ↓
Choose the appropriate service
 ↓
Implement
 ↓
Troubleshoot
 ↓
Secure
 ↓
Optimize
 ↓
Explain the tradeoffs
 ↓
Master


CURRENT STATUS:

Foundation Stress Test       🟢 Complete
CI/CD Module                 🟢 Complete
Cloud/AWS                    🔵 Starting next
Terraform                    ⚪ Not started
Kubernetes                   ⚪ Not started
Advanced Observability       🟡 Later
Production Architecture      ⚪ Later
Internship Preparation       🟡 Will begin before semester end


FINAL NOTE:

The objective is to become substantially stronger than a typical entry-level candidate through genuine technical depth, practical projects, troubleshooting ability, architecture reasoning, and strong fundamentals.

The roadmap will not be rushed simply to meet the February deadline.

Mastery remains the main goal.