 # ☁️ TOPIC: AWS
 > VPC networking, EC2 & Auto Scaling, load balancers, Route 53, IAM, S3, RDS, Lambda, API Gateway, CloudFront, ACM, monitoring & audit services, KMS, EventBridge, Secrets Manager, Systems Manager, ENI and Boto3.

<br>

---

### Q1. What is a VPC and what are its main components? How do you design one for production?

#### 📌 Short Explanation
The VPC is the foundation of every AWS architecture. Interviewers check whether you understand isolation, CIDR planning, and high availability.

#### ✅ Answer
A **VPC (Virtual Private Cloud)** is a logically isolated virtual network in AWS where you control IP ranges, subnets, routing, and security. A production VPC spans **at least two Availability Zones** with public and private subnets.

#### 📖 Detailed Explanation

**🔹 Main components**

| Component | Purpose |
|-----------|---------|
| **CIDR block** | IP range of the VPC (e.g., `10.0.0.0/16`) |
| **Subnets** | Segments of the VPC, each inside one AZ |
| **Route tables** | Decide where network traffic is directed |
| **Internet Gateway (IGW)** | Internet access for public subnets |
| **NAT Gateway** | Outbound-only internet for private subnets |
| **Security Groups / NACLs** | Instance-level and subnet-level firewalls |
| **VPC Endpoints** | Private access to AWS services |
| **Flow Logs** | Capture IP traffic metadata for troubleshooting and audit |

**🔹 Typical production design**
```text
VPC 10.0.0.0/16
├── AZ-a: public 10.0.1.0/24  | private-app 10.0.11.0/24 | private-db 10.0.21.0/24
└── AZ-b: public 10.0.2.0/24  | private-app 10.0.12.0/24 | private-db 10.0.22.0/24
```
- **Public:** ALB, NAT Gateway
- **Private-app:** EC2 / EKS / ECS workloads
- **Private-db:** RDS (no internet route)

**🔹 Best practices**
- Plan CIDRs so they **don't overlap** with on-premises or other VPCs (needed for peering/VPN)
- Enable **VPC Flow Logs**
- Use **VPC Endpoints** for S3/DynamoDB to avoid NAT cost
- Keep databases in private subnets

<br>

---

### Q2. How do Subnets work? How many IPs are reserved and how do you size them?

#### 📌 Short Explanation
Tests practical IP planning — a very common cause of production issues, especially with EKS.

#### ✅ Answer
A subnet is a range of the VPC CIDR placed in a **single Availability Zone**. AWS **reserves 5 IP addresses** in every subnet.

#### 📖 Detailed Explanation

**🔹 Reserved IPs (example `10.0.1.0/24`)**

| IP | Use |
|----|-----|
| `10.0.1.0` | Network address |
| `10.0.1.1` | VPC router |
| `10.0.1.2` | DNS server |
| `10.0.1.3` | Reserved for future use |
| `10.0.1.255` | Broadcast (not supported, but reserved) |

So a `/24` (256 IPs) gives **251 usable IPs**.

**🔹 Sizing tips**
- Smallest subnet: `/28` (16 IPs, 11 usable); largest: `/16`
- Leave room for growth; **EKS pods consume VPC IPs** (with VPC CNI), so give EKS subnets larger ranges (`/20` or bigger)
- ALB needs at least 8 free IPs per subnet
- A subnet **cannot span AZs** — create one per AZ for HA

**🔹 Public vs Private subnet**
A subnet is **public** only if its route table has `0.0.0.0/0 → Internet Gateway`. Otherwise it is private.

<br>

---

### Q3. Scenario: An EC2 instance in a public subnet cannot reach the internet. How do you troubleshoot using Route Tables?

#### 📌 Short Explanation
A classic networking scenario that tests the whole path: IGW, route table, public IP, SG, NACL.

#### ✅ Answer
Verify the **route table** has `0.0.0.0/0 → igw-xxxx`, the instance has a **public/Elastic IP**, the IGW is attached, and the **SG/NACL** allow traffic.

#### 📖 Detailed Explanation

**🔹 Route Tables basics**
- Every subnet is associated with exactly one route table (the **main** one if not explicitly associated)
- AWS uses **longest prefix match**; the `local` route (VPC CIDR) always exists and cannot be removed

| Destination | Target |
|-------------|--------|
| `10.0.0.0/16` | `local` |
| `0.0.0.0/0` | `igw-0abc123` |

**🔹 Checklist**
1. Is an **IGW attached** to the VPC?
2. Does the subnet's route table have `0.0.0.0/0 → IGW`?
3. Does the instance have a **public IPv4 or Elastic IP**?
4. Does the **Security Group** allow outbound traffic (default: allow all)?
5. Does the **NACL** allow outbound and **inbound ephemeral ports (1024–65535)**?
6. Is there an OS-level firewall or a wrong default gateway?

```bash
aws ec2 describe-route-tables --filters Name=association.subnet-id,Values=subnet-0123
aws ec2 describe-internet-gateways --filters Name=attachment.vpc-id,Values=vpc-0123
```

**🔹 Use Reachability Analyzer** to trace the path between source and destination and see exactly which component blocks it.

<br>

---

### Q4. What is the difference between an Internet Gateway and a NAT Gateway?

#### 📌 Short Explanation
One of the most frequently asked AWS networking questions.

#### ✅ Answer
- **Internet Gateway (IGW):** enables **two-way** internet communication for resources with public IPs.
- **NAT Gateway:** enables **outbound-only** internet access for private instances; the internet cannot initiate connections to them.

#### 📖 Detailed Explanation

| Aspect | Internet Gateway | NAT Gateway |
|--------|-----------------|-------------|
| **Direction** | Inbound + Outbound | Outbound only |
| **Used by** | Public subnets | Private subnets |
| **Lives in** | Attached to the VPC | Placed **inside a public subnet** |
| **Needs Elastic IP** | No (instance needs a public IP) | Yes (for public NAT gateway) |
| **Availability** | Highly available, managed by AWS | Scoped to **one AZ** |
| **Cost** | Free | Hourly charge + per-GB processing |
| **Route in table** | `0.0.0.0/0 → igw-xxx` | `0.0.0.0/0 → nat-xxx` |

**🔹 Traffic flow for a private instance**
```text
Private EC2 → Private route table → NAT Gateway (public subnet) → IGW → Internet
```

**🔹 NAT Gateway vs NAT Instance**
NAT Gateway is managed, scales automatically (up to 100 Gbps), and needs no patching. NAT Instance is self-managed but cheaper for tiny workloads.

<br>

---

### Q5. Scenario: Your NAT Gateway bill is very high, or private instances lose internet when one AZ fails. How do you fix it?

#### 📌 Short Explanation
Tests cost optimization and high-availability design around NAT.

#### ✅ Answer
Use **one NAT Gateway per AZ** (each AZ's private route table points to its own NAT) for resilience, and use **VPC Endpoints** to keep AWS-service traffic off the NAT to reduce cost.

#### 📖 Detailed Explanation

**🔹 Problem 1: Single NAT is an AZ-level single point of failure**
- If NAT in AZ-a fails, private subnets in AZ-b that route to it lose internet
- Also incurs **cross-AZ data transfer charges**

**Fix:** one NAT per AZ with separate route tables.

**🔹 Problem 2: High data processing cost**
Find top talkers with VPC Flow Logs, then:
- Add **S3 and DynamoDB Gateway Endpoints** (free)
- Add **Interface Endpoints** for ECR, CloudWatch, STS, SSM, etc.
- Pull container images from **ECR via endpoints**, not the internet
- Cache dependencies, use a proxy/mirror for package downloads

| Action | Effect |
|--------|--------|
| S3 Gateway Endpoint | Removes S3 traffic from NAT (free) |
| NAT per AZ | HA + avoids cross-AZ charges |
| Flow Logs analysis | Identify unexpected heavy traffic |

> 💡 For non-production environments, consider a single NAT (or scheduled scale-down) to save cost, accepting the lower availability.

<br>

---

### Q6. What is the difference between Security Groups and NACLs?

#### 📌 Short Explanation
A must-know comparison. It tests stateful vs stateless firewall concepts.

#### ✅ Answer
- **Security Group:** stateful firewall at the **instance/ENI** level, **allow rules only**.
- **NACL:** stateless firewall at the **subnet** level, supports **allow and deny** rules.

#### 📖 Detailed Explanation

| Feature | Security Group | Network ACL |
|---------|----------------|-------------|
| **Level** | ENI / instance | Subnet |
| **State** | **Stateful** (return traffic auto-allowed) | **Stateless** (must allow both directions) |
| **Rules** | Allow only | Allow and Deny |
| **Rule evaluation** | All rules evaluated together | In **number order**, first match wins |
| **Default** | Deny all inbound, allow all outbound | Default NACL allows all |
| **Reference other SGs** | ✅ Yes | ❌ No (CIDR only) |
| **Applies to** | Only attached resources | All resources in the subnet |

**🔹 Example: SG chaining (best practice)**
```text
ALB-SG:  inbound 443 from 0.0.0.0/0
App-SG:  inbound 8080 from ALB-SG
DB-SG:   inbound 3306 from App-SG
```

**🔹 When to use NACL**
- Block a **specific malicious IP/CIDR** (SGs can't deny)
- Add a coarse subnet-wide guardrail

> **Key takeaway:** SG = primary, fine-grained control. NACL = optional extra layer, mainly for explicit denies.

<br>

---

### Q7. Scenario: You cannot reach an application on port 80 on an EC2 instance. How do you tell whether the Security Group or NACL is the cause?

#### 📌 Short Explanation
Tests systematic troubleshooting across layers instead of guessing.

#### ✅ Answer
Work from the instance outward: app listening → OS firewall → Security Group → NACL → route table → public IP/DNS. Use **VPC Flow Logs** (`ACCEPT`/`REJECT`) to identify the blocking layer.

#### 📖 Detailed Explanation

**🔹 Step 1: Is the app listening?**
```bash
ss -tulnp | grep :80
curl -I localhost
```

**🔹 Step 2: Security Group** — inbound rule for TCP 80 from the client's source?

**🔹 Step 3: NACL** — remember it's stateless:
- Inbound: allow TCP 80
- Outbound: allow **ephemeral ports 1024–65535** for the response
- Check no lower-numbered **DENY** rule matches first

**🔹 Step 4: Route table / IGW / public IP** (see earlier scenario)

**🔹 Step 5: Read Flow Logs**
```text
... 203.0.113.5 10.0.1.25 51234 80 6 ... REJECT OK
```
- `REJECT` on inbound only with SG fine → likely NACL
- Traffic accepted inbound but reply rejected → NACL missing ephemeral outbound rule

| Symptom | Likely cause |
|---------|-------------|
| `Connection timed out` | SG/NACL/route blocking packets |
| `Connection refused` | Reached the host, but nothing is listening on the port |

<br>

---

### Q8. What is VPC Peering? What are its limitations, and how does it differ from Transit Gateway?

#### 📌 Short Explanation
Tests multi-VPC connectivity design knowledge.

#### ✅ Answer
**VPC Peering** is a private, one-to-one network connection between two VPCs (same or different account/region). It is **non-transitive** and requires **non-overlapping CIDRs**.

#### 📖 Detailed Explanation

**🔹 Setup steps**
1. Requester creates a peering connection; accepter accepts it
2. Add **routes** in both VPCs' route tables to the peer CIDR
3. Update **Security Groups** (and NACLs) to allow traffic

**🔹 Limitations**
- ❌ **No transitive routing** (A↔B and B↔C does not give A↔C)
- ❌ **No overlapping CIDRs**
- ❌ No edge-to-edge routing (can't use the peer's IGW, NAT, VPN, or Direct Connect)
- Becomes unmanageable at scale (full mesh = N×(N−1)/2 connections)

**🔹 Peering vs Transit Gateway**

| Aspect | VPC Peering | Transit Gateway |
|--------|-------------|-----------------|
| **Topology** | Point-to-point mesh | Hub-and-spoke |
| **Transitive routing** | ❌ No | ✅ Yes |
| **Scale** | Few VPCs | Hundreds of VPCs, VPNs, Direct Connect |
| **Cost** | No hourly fee (data transfer charged) | Hourly attachment + per-GB fee |
| **Bandwidth** | No bottleneck | Per-attachment limits |

```bash
aws ec2 create-vpc-peering-connection --vpc-id vpc-aaa --peer-vpc-id vpc-bbb
aws ec2 accept-vpc-peering-connection --vpc-peering-connection-id pcx-123
```

<br>

---

### Q9. What are VPC Endpoints? Explain Gateway vs Interface endpoints.

#### 📌 Short Explanation
Tests security and cost awareness — keeping AWS service traffic on the AWS network.

#### ✅ Answer
**VPC Endpoints** let resources in a VPC reach AWS services **privately**, without going through the internet, NAT, or IGW. There are **Gateway endpoints** (S3, DynamoDB) and **Interface endpoints** (powered by AWS PrivateLink).

#### 📖 Detailed Explanation

| Feature | Gateway Endpoint | Interface Endpoint |
|---------|------------------|--------------------|
| **Services** | **S3 and DynamoDB only** | Most AWS services (ECR, SSM, STS, SQS, Secrets Manager…), plus your own PrivateLink services |
| **Implementation** | Route table entry (prefix list) | **ENI** with private IP in your subnet |
| **Cost** | **Free** | Hourly + per-GB charge |
| **Security control** | Endpoint policy | Endpoint policy + **Security Group** |
| **Reachable from on-prem** | ❌ No | ✅ Yes (via VPN/Direct Connect) |

**🔹 Example: restrict an S3 bucket to a VPC endpoint**
```json
{
  "Effect": "Deny",
  "Principal": "*",
  "Action": "s3:*",
  "Resource": ["arn:aws:s3:::my-bucket", "arn:aws:s3:::my-bucket/*"],
  "Condition": { "StringNotEquals": { "aws:SourceVpce": "vpce-0abc123" } }
}
```

**🔹 Scenario: private subnet with no internet needs SSM Session Manager**
Create Interface endpoints for `ssm`, `ssmmessages`, and `ec2messages` with Private DNS enabled.

> **Key takeaway:** Endpoints improve **security** (no public path) and reduce **NAT costs**.

<br>

---

### Q10. Explain Elastic Network Interfaces (ENI). How are they different from Elastic IPs, and where are they useful?

#### 📌 Short Explanation
ENIs are behind almost every AWS networking feature. Interviewers test whether you know their attributes and practical uses.

#### ✅ Answer
An **ENI** is a virtual network card in a VPC. It carries a primary private IP, optional secondary IPs, a MAC address, and Security Groups, and can be attached to an EC2 instance. An **Elastic IP** is a static public IPv4 address that can be associated with an ENI.

#### 📖 Detailed Explanation

**🔹 ENI attributes**
- Primary private IPv4 + optional secondary IPs, IPv6 addresses
- One or more Security Groups
- MAC address and source/destination check flag
- Lives in **one subnet / one AZ**

**🔹 Key behaviors**
- The **primary ENI (eth0)** can't be detached
- Secondary ENIs can be **hot-attached, detached, and moved** between instances **in the same AZ**
- The number of ENIs and IPs per instance depends on the **instance type**

**🔹 ENI vs Elastic IP**

| Aspect | ENI | Elastic IP |
|--------|-----|-----------|
| **What** | Virtual network card | Static public IPv4 address |
| **Scope** | Subnet / AZ | Region |
| **Holds** | Private IPs, SGs, MAC | Just the public address |
| **Moves** | Between instances in same AZ | Between ENIs/instances |

**🔹 Use cases**
- **Failover:** move a secondary ENI (with its IP) to a standby instance
- **Management network:** separate admin traffic from application traffic
- **Licensing** tied to a MAC address
- **Network appliances/NAT instances** (disable source/dest check)
- Interface VPC endpoints, Lambda in VPC, ALB, and RDS all use ENIs under the hood

```bash
aws ec2 attach-network-interface --network-interface-id eni-0abc --instance-id i-0123 --device-index 1
```

<br>

---

### Q11. How does Route 53 work? Explain routing policies and the difference between Alias and CNAME records.

#### 📌 Short Explanation
DNS is critical for availability and traffic management; routing policies are a frequent interview topic.

#### ✅ Answer
**Route 53** is AWS's highly available DNS service that also supports health checks and domain registration. Routing policies decide *how* DNS answers are returned.

#### 📖 Detailed Explanation

**🔹 Routing policies**

| Policy | Behavior | Use case |
|--------|----------|----------|
| **Simple** | One record, one or multiple values | Single resource |
| **Weighted** | Split traffic by weight | Canary / A-B testing |
| **Latency** | Lowest-latency region | Multi-region apps |
| **Failover** | Primary/secondary with health checks | Active-passive DR |
| **Geolocation** | By user's country/continent | Compliance, localization |
| **Geoproximity** | By distance, with bias | Shift traffic between regions |
| **Multivalue** | Up to 8 healthy records | Basic DNS load distribution |

**🔹 Alias vs CNAME**

| Aspect | Alias | CNAME |
|--------|-------|-------|
| **Zone apex (`example.com`)** | ✅ Allowed | ❌ Not allowed |
| **Cost** | **Free** queries to AWS resources | Charged |
| **Targets** | AWS resources (ALB, CloudFront, S3 website, API Gateway) | Any DNS name |
| **Health check** | Can evaluate target health | Separate health check |

**🔹 Scenario: active-passive DR**
Create a primary record (ALB in us-east-1) with a health check and a secondary record (ALB in us-west-2) with **Failover** routing. If the health check fails, Route 53 returns the secondary.

**🔹 Public vs Private hosted zone**
A **private hosted zone** resolves names only inside associated VPCs (e.g., `db.internal.example.com`).

<br>

---

### Q12. Explain EC2 purchasing options and how you choose between them.

#### 📌 Short Explanation
Tests cost-optimization knowledge, which is a major part of real AWS work.

#### ✅ Answer
Use **On-Demand** for unpredictable workloads, **Savings Plans / Reserved Instances** for steady baseline usage, and **Spot** for fault-tolerant, interruptible workloads.

#### 📖 Detailed Explanation

| Option | Commitment | Discount (approx.) | Best for |
|--------|-----------|--------------------|----------|
| **On-Demand** | None | 0% | New, spiky, short-term workloads |
| **Savings Plans** | 1 or 3 years of $/hour spend | Up to ~72% | Steady usage, flexible across instance families |
| **Reserved Instances** | 1 or 3 years for specific config | Up to ~72% | Predictable workloads (e.g., databases) |
| **Spot** | None | Up to ~90% | Batch, CI runners, stateless workers, containers |
| **Dedicated Hosts** | Optional | — | BYOL licensing, compliance |

**🔹 Spot best practices**
- Handle the **2-minute interruption notice**
- Diversify across **instance types and AZs**
- Use an **Auto Scaling Group with mixed instances** (On-Demand base + Spot)

**🔹 Instance families (quick reference)**
- `t` / `m` — general purpose · `c` — compute · `r` — memory · `i` — storage · `g/p` — GPU

> 💡 A common real-world pattern: **Savings Plans for the baseline, On-Demand for spikes, Spot for batch and non-critical tiers**.

<br>

---

### Q13. Scenario: An EC2 instance fails status checks or becomes unreachable. How do you troubleshoot?

#### 📌 Short Explanation
Tests whether you understand the two types of status checks and the recovery options.

#### ✅ Answer
Identify whether it is a **System status check** (AWS hardware/network issue) or an **Instance status check** (OS/config issue), then recover accordingly.

#### 📖 Detailed Explanation

| Check | Detects | Typical fix |
|-------|---------|-------------|
| **System status** | Host hardware, power, network problems | **Stop and start** (moves to new hardware) or enable **auto-recovery** |
| **Instance status** | Kernel panic, bad network config, full disk, OOM | Reboot, fix config, or mount the volume on a rescue instance |

**🔹 Steps**
1. Check **EC2 Console → Status checks** and **Get system log / instance screenshot**
2. Check CloudWatch metrics (`CPUUtilization`, `StatusCheckFailed`)
3. Try **Session Manager** or **EC2 Instance Connect**
4. If the OS is broken (bad `fstab`, full disk, wrong SSH config):
   - Stop instance → detach root EBS → attach to a **rescue instance** → fix → reattach → start
5. Create an **AMI/snapshot** before risky changes

**🔹 Prevent**
- CloudWatch alarm on `StatusCheckFailed_System` with action **`recover`**
- Auto Scaling Group health checks to replace bad instances automatically

| Action | Effect |
|--------|--------|
| **Reboot** | Same host, same IPs, data kept |
| **Stop/Start** | New host, new public IP (unless EIP), EBS data kept |
| **Terminate** | Instance deleted (root EBS deleted by default) |

<br>

---

### Q14. What is an AMI? Explain the Golden AMI approach and the difference between an AMI and a snapshot.

#### 📌 Short Explanation
Tests your understanding of immutable infrastructure and image management.

#### ✅ Answer
An **AMI (Amazon Machine Image)** is a template containing the OS, software, and block-device mappings used to launch EC2 instances. A **Golden AMI** is a pre-hardened, pre-baked image used as the standard base.

#### 📖 Detailed Explanation

**🔹 AMI vs Snapshot**

| Aspect | AMI | EBS Snapshot |
|--------|-----|-------------|
| **What** | Launchable image (metadata + snapshots) | Point-in-time backup of one volume |
| **Launch an instance?** | ✅ Yes | ❌ No (create volume or AMI first) |
| **Contains** | Root + data volume snapshots, launch permissions, mappings | Just volume data |

**🔹 Golden AMI workflow**
```text
Base AMI → Packer/Image Builder (patch, harden, install agents) → Test → Golden AMI → Launch Template → ASG
```

**🔹 Key facts**
- AMIs are **regional**; copy to other regions with `copy-image`
- Can be shared with other accounts or made public
- **Pre-baking** speeds up launches vs installing everything in `user_data`
- Encrypted AMIs require **KMS key access** to be shared cross-account
- Use **EC2 Image Builder** or **Packer** for automated, repeatable builds

```bash
aws ec2 create-image --instance-id i-0123 --name "web-golden-2026-10-11" --no-reboot
aws ec2 copy-image --source-image-id ami-0abc --source-region us-east-1 --region ap-south-1 --name web-golden
```

<br>

---

### Q15. What is the difference between Launch Templates and Launch Configurations?

#### 📌 Short Explanation
Auto Scaling groups need a blueprint to launch instances; this checks you know the current best practice.

#### ✅ Answer
**Launch Templates** are the modern, versioned, feature-rich replacement for the legacy **Launch Configurations**, which are deprecated and no longer receive new features.

#### 📖 Detailed Explanation

| Feature | Launch Template | Launch Configuration |
|---------|-----------------|----------------------|
| **Versioning** | ✅ Yes | ❌ Immutable, no versions |
| **Edit** | Create new versions | Must create a new one |
| **Mixed On-Demand + Spot** | ✅ Yes | ❌ No |
| **Multiple instance types** | ✅ Yes | ❌ No |
| **Newer features** (T3 unlimited, IMDSv2 options, placement, etc.) | ✅ Yes | ❌ Limited |
| **Use outside ASG** (`run-instances`, Spot Fleet) | ✅ Yes | ❌ ASG only |
| **Status** | Recommended | Deprecated |

**🔹 What a template defines**
AMI, instance type, key pair, Security Groups, IAM instance profile, `user_data`, EBS mappings, tags, metadata options.

**🔹 Rolling out a new AMI**
1. Create a **new template version** with the new AMI
2. Update the ASG to use the new version (`$Latest` or a fixed number)
3. Run **Instance Refresh** for a rolling replacement

```bash
aws autoscaling start-instance-refresh --auto-scaling-group-name web-asg \
  --preferences '{"MinHealthyPercentage":90,"InstanceWarmup":120}'
```

<br>

---

### Q16. Explain Auto Scaling policies. Scenario: your ASG isn't scaling out under load — what do you check?

#### 📌 Short Explanation
Tests both the theory of scaling policies and practical troubleshooting.

#### ✅ Answer
ASG supports **target tracking, step, simple, scheduled, and predictive** scaling. If it isn't scaling, check the alarm/metric, max capacity, cooldown/warmup, launch failures, and quotas.

#### 📖 Detailed Explanation

**🔹 Scaling policy types**

| Policy | How it works | Example |
|--------|-------------|---------|
| **Target tracking** | Maintains a metric at a target | Keep average CPU at 50% |
| **Step scaling** | Different adjustments for alarm severity | +2 if CPU > 70%, +4 if > 90% |
| **Simple scaling** | One adjustment per alarm, with cooldown | Legacy |
| **Scheduled** | Scale at specific times | Scale up before 9 AM sale |
| **Predictive** | ML forecasts load from history | Recurring daily patterns |

**🔹 Troubleshooting checklist**
- ✅ Is **MaxSize** already reached?
- ✅ Is the CloudWatch **alarm** in `ALARM` state? Is the metric reporting? (memory needs the CloudWatch agent)
- ✅ Is it in **cooldown / instance warmup**?
- ✅ Check **Activity history** for errors: `InsufficientInstanceCapacity`, bad AMI, missing key pair, wrong Security Group, **service quota exceeded**
- ✅ Does the launch template reference a **deleted AMI/SG/subnet**?
- ✅ Is the subnet **out of free IPs**?

```bash
aws autoscaling describe-scaling-activities --auto-scaling-group-name web-asg --max-items 5
```

**🔹 Health checks:** use **ELB health checks** (not only EC2) so unhealthy app instances get replaced.

<br>

---

### Q17. Explain Elastic Load Balancers. What is the difference between ALB, NLB, and Gateway Load Balancer?

#### 📌 Short Explanation
A must-know for any AWS interview — choosing the right load balancer.

#### ✅ Answer
- **ALB** — Layer 7 (HTTP/HTTPS), content-based routing
- **NLB** — Layer 4 (TCP/UDP/TLS), ultra-low latency and static IPs
- **GWLB** — Layer 3 gateway for inserting third-party virtual appliances (firewalls)
- **CLB** — legacy, avoid for new workloads

#### 📖 Detailed Explanation

| Feature | ALB | NLB |
|---------|-----|-----|
| **OSI layer** | 7 | 4 |
| **Protocols** | HTTP, HTTPS, gRPC, WebSocket | TCP, UDP, TLS |
| **Routing** | Host, path, header, query string | Port/protocol based |
| **Static / Elastic IP** | ❌ (use Global Accelerator) | ✅ Yes, per AZ |
| **Performance** | High | Millions of requests/sec, very low latency |
| **Preserves client IP** | Via `X-Forwarded-For` header | Yes (for instance targets) |
| **Security Groups** | ✅ Yes | ✅ Yes (supported on newer NLBs) |
| **WAF integration** | ✅ Yes | ❌ No |
| **Targets** | Instances, IPs, Lambda | Instances, IPs, ALB |

**🔹 When to choose**
- **ALB:** web apps, microservices, path-based routing (`/api` → service A, `/web` → service B), Kubernetes Ingress
- **NLB:** non-HTTP protocols, gaming, IoT, need static IPs, extreme throughput, PrivateLink services
- **GWLB:** inline security inspection appliances

**🔹 Features common to ELB:** cross-zone load balancing, SSL termination (ACM), health checks, connection draining (deregistration delay), access logs.

<br>

---

### Q18. What are Target Groups? Scenario: ALB returns 502, 503, or 504 errors — how do you troubleshoot?

#### 📌 Short Explanation
Tests your understanding of ALB internals and the meaning of each error code.

#### ✅ Answer
A **Target Group** is a set of targets (instances, IPs, Lambda) that a load balancer routes to, with its own **health check** configuration. Each 5xx code points to a different layer of the problem.

#### 📖 Detailed Explanation

**🔹 Target group config**
- Target type: `instance`, `ip`, `lambda`, `alb`
- Protocol/port, **health check** (path, interval, thresholds, matcher codes)
- Deregistration delay (connection draining), stickiness, load-balancing algorithm

**🔹 Error codes**

| Code | Meaning | Common causes and fixes |
|------|---------|-------------------------|
| **502 Bad Gateway** | Target sent an invalid response or closed the connection | App crashed, protocol mismatch (HTTP vs HTTPS), target **keep-alive timeout < ALB idle timeout** (set app keep-alive higher than 60s) |
| **503 Service Unavailable** | **No healthy targets** (or none registered) | Health check path wrong, SG blocks ALB → target, app down |
| **504 Gateway Timeout** | Target didn't respond within the idle timeout | Slow app/DB, SG/NACL blocking, increase idle timeout or fix slow query |

**🔹 Targets showing `unhealthy`?**
1. Is the **health check path/port** correct and returning the expected code (e.g., `200`)?
2. Does the **target's SG allow inbound from the ALB's SG**?
3. Is the app listening on the right port and bound to `0.0.0.0`, not `127.0.0.1`?
4. Check ALB **access logs** and target logs

```bash
aws elbv2 describe-target-health --target-group-arn <arn>
```

<br>

---

### Q19. Design a highly available, scalable 3-tier web application on AWS.

#### 📌 Short Explanation
The most common architecture scenario. It combines networking, compute, database, and security knowledge.

#### ✅ Answer
Use a multi-AZ VPC with **ALB (public) → EC2/ECS/EKS in an Auto Scaling Group (private) → RDS Multi-AZ (isolated)**, with Route 53, CloudFront, WAF, and CloudWatch around it.

#### 📖 Detailed Explanation

```text
Users → Route 53 → CloudFront (+ WAF) → ALB (public subnets, 2+ AZs)
                                          ↓
                         Auto Scaling Group (private app subnets)
                                          ↓
                    RDS Multi-AZ + Read Replicas (private DB subnets)
                    ElastiCache (optional)    S3 (static assets / backups)
```

| Tier | AWS Services | HA technique |
|------|--------------|--------------|
| **Edge** | Route 53, CloudFront, WAF, ACM | Global edge caching, DDoS protection (Shield) |
| **Web / App** | ALB, EC2 ASG (or ECS/EKS) | Multi-AZ, health checks, auto scaling |
| **Data** | RDS Multi-AZ, ElastiCache, S3 | Automatic failover, replicas, cache |
| **Network** | VPC, NAT per AZ, Endpoints | No single point of failure |
| **Security** | SGs chaining, IAM roles, KMS, Secrets Manager | Least privilege, encryption |
| **Ops** | CloudWatch, CloudTrail, Config, SSM | Monitoring, audit, patching |

**🔹 Design principles to mention**
- No SSH to servers; use **Session Manager**
- Instances use **IAM roles**, not access keys
- **Stateless** app tier (sessions in ElastiCache/DynamoDB)
- **Infrastructure as Code** (Terraform) and golden AMIs
- **Backups + DR:** cross-region snapshots, Route 53 failover
- Cost control: right-sizing, Savings Plans, S3 lifecycle policies

<br>

---

### Q20. Explain IAM users, groups, roles, and policies. What is the difference between a User and a Role?

#### 📌 Short Explanation
IAM is the most important AWS security service. This question tests identity fundamentals.

#### ✅ Answer
- **User** — long-term identity for a person/app with permanent credentials
- **Group** — collection of users sharing permissions
- **Role** — an identity with **temporary credentials**, assumed by users, services, or other accounts
- **Policy** — JSON document defining permissions

#### 📖 Detailed Explanation

| Aspect | IAM User | IAM Role |
|--------|----------|----------|
| **Credentials** | Long-term (password, access keys) | Temporary (STS tokens) |
| **Assumed by** | One specific person/app | Anyone/anything trusted (EC2, Lambda, other accounts, federated users) |
| **Best practice** | Avoid for workloads; prefer SSO | ✅ Preferred for workloads |

**🔹 Policy types**

| Type | Purpose |
|------|---------|
| **Identity-based** | Attached to users/groups/roles |
| **Resource-based** | Attached to resources (S3 bucket policy, KMS key policy) |
| **Trust policy** | Defines *who can assume* a role |
| **Permission boundary** | Maximum permissions an identity can have |
| **SCP (Organizations)** | Guardrails for entire accounts |
| **Session policy** | Limits permissions of an assumed session |

**🔹 Example policy (least privilege)**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["s3:GetObject", "s3:PutObject"],
    "Resource": "arn:aws:s3:::my-app-bucket/uploads/*"
  }]
}
```

**🔹 Evaluation logic**
1. Default is **implicit deny**
2. An **explicit Deny always wins**
3. Otherwise, an explicit Allow is needed (and must not be blocked by SCP / boundary)

**🔹 Best practices:** enable MFA, no root usage, least privilege, rotate keys, use IAM Identity Center (SSO).

<br>

---

### Q21. Scenario: An application on EC2 needs to access S3 securely, and another account's resources too. How do you do it without hardcoding keys?

#### 📌 Short Explanation
Tests IAM roles, instance profiles, and cross-account access with STS AssumeRole.

#### ✅ Answer
Attach an **IAM role via an instance profile** to EC2 for S3 access. For cross-account access, create a role in the target account that **trusts** the source account, and have the app call **`sts:AssumeRole`**.

#### 📖 Detailed Explanation

**🔹 Same account: EC2 → S3**
1. Create an IAM role with trust policy for `ec2.amazonaws.com`
2. Attach a least-privilege S3 policy
3. Attach the role to the instance (instance profile)
4. The SDK/CLI automatically retrieves temporary credentials from the metadata service (use **IMDSv2**)

**🔹 Cross-account**

*Account B (target) — role trust policy:*
```json
{
  "Effect": "Allow",
  "Principal": { "AWS": "arn:aws:iam::111111111111:role/app-role" },
  "Action": "sts:AssumeRole"
}
```
*Account A (source) — role must have `sts:AssumeRole` permission on that role ARN.*

```bash
aws sts assume-role --role-arn arn:aws:iam::222222222222:role/cross-acct-role --role-session-name demo
```

**🔹 Troubleshooting `AccessDenied`**
- Check identity policy, resource policy, **SCP**, permission boundary, and **explicit deny**
- For KMS-encrypted objects, the role also needs **KMS permissions**
- Use **IAM Policy Simulator** and CloudTrail to find the exact denied action

> **Key takeaway:** **Never embed access keys** in code or AMIs; use roles with temporary credentials.

<br>

---

### Q22. Explain Amazon S3 storage classes, versioning, and lifecycle policies.

#### 📌 Short Explanation
S3 is the most used AWS service. Interviewers test durability, cost optimization, and data protection.

#### ✅ Answer
S3 offers multiple **storage classes** for different access patterns. **Versioning** protects against accidental deletion/overwrite, and **lifecycle rules** automatically transition or expire objects to save cost.

#### 📖 Detailed Explanation

**🔹 Storage classes**

| Class | Use case | Retrieval |
|-------|----------|-----------|
| **Standard** | Frequently accessed data | Instant |
| **Intelligent-Tiering** | Unknown or changing access patterns | Instant (auto-tiers) |
| **Standard-IA** | Infrequent access, multi-AZ | Instant, retrieval fee |
| **One Zone-IA** | Infrequent, re-creatable data, single AZ | Instant |
| **Glacier Instant Retrieval** | Archive needing millisecond access | Instant |
| **Glacier Flexible Retrieval** | Archive (minutes to hours) | Minutes–hours |
| **Glacier Deep Archive** | Long-term compliance archive | ~12–48 hours |

All classes provide **11 nines (99.999999999%) durability** (data is spread across AZs, except One Zone).

**🔹 Versioning**
- Keeps every version of an object; a delete adds a **delete marker**
- Enables recovery, and is needed for **replication** and **MFA Delete**

**🔹 Lifecycle example**
```json
{
  "Rules": [{
    "ID": "logs-archive",
    "Status": "Enabled",
    "Filter": { "Prefix": "logs/" },
    "Transitions": [
      { "Days": 30, "StorageClass": "STANDARD_IA" },
      { "Days": 90, "StorageClass": "GLACIER" }
    ],
    "Expiration": { "Days": 365 },
    "NoncurrentVersionExpiration": { "NoncurrentDays": 30 }
  }]
}
```

**🔹 Other features:** Cross-Region Replication, Object Lock (WORM), Transfer Acceleration, multipart upload (recommended for files > 100 MB), pre-signed URLs, S3 is **strongly consistent** for reads after writes.

<br>

---

### Q23. Scenario: How do you secure an S3 bucket, and how do you troubleshoot `AccessDenied` errors?

#### 📌 Short Explanation
Misconfigured S3 buckets are a leading cause of data leaks. Tests both security and troubleshooting skills.

#### ✅ Answer
Enable **Block Public Access**, enforce **encryption and TLS**, use **least-privilege IAM/bucket policies**, turn on **versioning and logging**. For `AccessDenied`, check every policy layer.

#### 📖 Detailed Explanation

**🔹 Security checklist**
- ✅ **S3 Block Public Access** (account and bucket level)
- ✅ **Default encryption** (SSE-S3 or SSE-KMS)
- ✅ Bucket policy denying non-TLS requests:
```json
{
  "Effect": "Deny",
  "Principal": "*",
  "Action": "s3:*",
  "Resource": ["arn:aws:s3:::my-bucket", "arn:aws:s3:::my-bucket/*"],
  "Condition": { "Bool": { "aws:SecureTransport": "false" } }
}
```
- ✅ Access via **IAM roles / VPC endpoints**; avoid long-lived keys
- ✅ **Versioning + MFA Delete + Object Lock** for critical data
- ✅ **Server access logs / CloudTrail data events**
- ✅ Monitor with **AWS Config** (`s3-bucket-public-read-prohibited`) and **Macie** for sensitive data

**🔹 `AccessDenied` troubleshooting**

| Check | Details |
|-------|---------|
| **IAM policy** | Does the role allow the action on the correct ARN (`bucket` vs `bucket/*`)? |
| **Bucket policy** | Any explicit Deny (VPC endpoint, IP, TLS conditions)? |
| **Block Public Access / ACLs** | Blocking public or cross-account access? |
| **KMS** | SSE-KMS object needs `kms:Decrypt` on the key |
| **SCP / boundary** | Org-level restriction? |
| **Cross-account** | Needs permission on **both** sides |
| **Object ownership** | Uploaded by another account without bucket-owner-enforced? |

<br>

---

### Q24. What is the difference between S3, EBS, and EFS?

#### 📌 Short Explanation
Tests your ability to pick the right storage type.

#### ✅ Answer
- **S3** — object storage (HTTP API, unlimited)
- **EBS** — block storage for a single EC2 instance (like a disk)
- **EFS** — managed shared NFS file system for many instances

#### 📖 Detailed Explanation

| Feature | S3 | EBS | EFS |
|---------|----|-----|-----|
| **Type** | Object | Block | File (NFS) |
| **Access** | HTTP/S API | Attached to one instance (Multi-Attach for io1/io2 only) | Mounted by thousands of instances/containers |
| **Scope** | Regional | **Single AZ** | Regional (multi-AZ) |
| **Capacity** | Unlimited | Provisioned (up to 64 TiB) | Elastic, grows automatically |
| **Use cases** | Backups, static sites, data lakes, logs | Boot volumes, databases | Shared content, CMS, home dirs |
| **Pricing** | Per GB stored + requests | Per GB provisioned | Per GB used |

**🔹 EBS volume types**
- `gp3` — default general purpose (baseline 3,000 IOPS, tunable)
- `io2` — high-performance databases
- `st1` / `sc1` — throughput/cold HDD

**🔹 Scenario:** "Need a shared folder for 10 web servers" → **EFS**. "Need a fast disk for MySQL on EC2" → **EBS gp3/io2**. "Store user uploads and serve them" → **S3 + CloudFront**.

<br>

---

### Q25. What is the difference between RDS Multi-AZ and Read Replicas?

#### 📌 Short Explanation
One of the most common RDS questions. Candidates often confuse high availability with read scaling.

#### ✅ Answer
- **Multi-AZ** is for **high availability and failover** (synchronous standby in another AZ).
- **Read Replicas** are for **read scalability** (asynchronous copies serving read traffic).

#### 📖 Detailed Explanation

| Aspect | Multi-AZ | Read Replica |
|--------|----------|--------------|
| **Purpose** | HA / disaster recovery | Scale reads, reporting |
| **Replication** | **Synchronous** | **Asynchronous** |
| **Serves traffic?** | ❌ Standby not readable (classic Multi-AZ instance) | ✅ Yes, read-only endpoint |
| **Failover** | Automatic (typically 60–120 seconds) | Manual **promotion** |
| **Location** | Same region, different AZ | Same region, **cross-AZ or cross-region** |
| **Endpoint** | Same DNS endpoint (CNAME flips) | Separate endpoint per replica |
| **Backups** | Taken from standby (less impact) | Independent |

> 💡 Newer **Multi-AZ DB cluster** deployments have two readable standbys, combining HA and read scaling.

**🔹 Failover triggers:** instance/AZ failure, OS patching, instance type change, manual reboot with failover.

> **Key takeaway:** Multi-AZ ≠ read scaling. Use both for production: Multi-AZ for resilience, replicas for performance.

<br>

---

### Q26. Scenario: Your application is read-heavy and the RDS database is slow. How do Read Replicas help, and what do you need to change?

#### 📌 Short Explanation
Tests practical database scaling and awareness of replication lag.

#### ✅ Answer
Create one or more **Read Replicas**, route **read queries** to them (via a reader endpoint or app-level routing), and keep writes on the primary. Monitor **replica lag**.

#### 📖 Detailed Explanation

**🔹 Steps**
1. Create a replica
```bash
aws rds create-db-instance-read-replica --db-instance-identifier mydb-replica1 --source-db-instance-identifier mydb
```
2. Update the app: **write → primary endpoint**, **read → replica endpoint(s)** (or use a read/write splitting library or proxy)
3. Monitor CloudWatch `ReplicaLag`
4. Add caching (**ElastiCache**) and indexes before scaling out further

**🔹 Considerations**
- Replication is **asynchronous** → possible **stale reads**; critical reads right after a write should use the primary
- Replicas can be **promoted** to standalone databases (used in DR and migrations)
- **Cross-region replicas** help DR and low-latency global reads
- Replicas can have different instance sizes and their own Multi-AZ setting
- **Aurora** supports up to 15 replicas with a single **reader endpoint** and typically much lower lag

**🔹 Other ways to improve performance:** right-size the instance, use Provisioned IOPS, enable Performance Insights, optimize slow queries, and use RDS Proxy for connection management.

<br>

---

### Q27. What is RDS Proxy and when should you use it?

#### 📌 Short Explanation
Tests knowledge of connection management, especially for serverless apps connecting to relational databases.

#### ✅ Answer
**RDS Proxy** is a fully managed database proxy that **pools and shares connections**, reduces database load, and speeds up failover. It is particularly useful with **Lambda** and apps that open many short-lived connections.

#### 📖 Detailed Explanation

**🔹 Problem it solves**
Lambda can scale to hundreds of concurrent executions, each opening its own DB connection and exhausting `max_connections`, causing `Too many connections` errors and high CPU.

**🔹 Benefits**
- **Connection pooling and multiplexing**
- **Faster failover** (proxy keeps connections and reduces failover impact)
- **IAM authentication** and credentials stored in **Secrets Manager**
- Enforces TLS

**🔹 Architecture**
```text
Lambda / App → RDS Proxy (in VPC) → RDS / Aurora (Multi-AZ)
```

**🔹 Notes**
- Runs inside your **VPC** (not publicly accessible); the app connects to the **proxy endpoint**
- Supports MySQL, PostgreSQL, MariaDB, and SQL Server engines
- Extra cost per vCPU of the DB instance
- **Pinning:** certain session-level features can pin a connection to a client, reducing multiplexing benefits

| Without RDS Proxy | With RDS Proxy |
|-------------------|----------------|
| Each Lambda opens a connection | Shared pool of connections |
| Failover drops connections | Faster, smoother failover |
| DB credentials in env vars | Secrets Manager + IAM auth |

<br>

---

### Q28. Explain AWS Lambda. Scenario: your Lambda function is timing out or being throttled — what do you do?

#### 📌 Short Explanation
Tests your understanding of serverless limits, concurrency, and cold starts.

#### ✅ Answer
**Lambda** runs code without managing servers, billed per request and execution duration. Timeouts and throttling are solved by tuning **timeout/memory**, fixing slow dependencies, and managing **concurrency**.

#### 📖 Detailed Explanation

**🔹 Key limits (verify the current values in the AWS docs)**
- Max timeout: **15 minutes**
- Memory: 128 MB – 10 GB (CPU scales with memory)
- Default concurrency: **1,000 per region** (can be raised)
- Synchronous payload: **6 MB**
- `/tmp` storage: up to 10 GB

**🔹 Timeouts — causes and fixes**
- Slow downstream calls (DB, API) → add timeouts, caching, RDS Proxy
- Too little memory → **increase memory** (more CPU)
- Lambda in a VPC without NAT/endpoints can't reach the internet → add NAT or VPC endpoints
- Long processing → split work, use **Step Functions** or SQS

**🔹 Throttling (`429 TooManyRequestsException`)**

| Concept | Meaning |
|---------|---------|
| **Reserved concurrency** | Guarantees (and caps) concurrency for one function |
| **Provisioned concurrency** | Pre-warmed environments, eliminates cold starts |
| **Account limit** | Shared by all functions in the region → request an increase |

**🔹 Cold starts**
Mitigate with provisioned concurrency, smaller packages, lighter runtimes, avoiding heavy init code, and SnapStart (supported runtimes).

**🔹 Error handling for async invocations:** retries (2 by default), **DLQ / on-failure destinations**, and idempotent code.

```bash
aws lambda get-function-concurrency --function-name my-func
aws logs tail /aws/lambda/my-func --follow
```

<br>

---

### Q29. What is API Gateway? Explain REST API vs HTTP API and how you secure and throttle APIs.

#### 📌 Short Explanation
Tests knowledge of API management, which is often paired with Lambda.

#### ✅ Answer
**API Gateway** is a managed service to create, publish, secure, and monitor APIs. It offers **REST APIs**, cheaper/faster **HTTP APIs**, and **WebSocket APIs**.

#### 📖 Detailed Explanation

| Feature | REST API | HTTP API |
|---------|----------|----------|
| **Cost** | Higher | ~70% cheaper |
| **Latency** | Higher | Lower |
| **API keys / usage plans** | ✅ Yes | ❌ No |
| **Request validation, caching, WAF** | ✅ Yes | ❌ Limited |
| **JWT authorizers (OIDC/OAuth2)** | Via Lambda/Cognito | ✅ Native |
| **Best for** | Full-featured, enterprise APIs | Simple, low-cost proxy to Lambda/HTTP |

**🔹 Authentication options**
- **IAM (SigV4)** — service-to-service
- **Cognito User Pools / JWT** — end users
- **Lambda authorizer** — custom logic
- **mTLS, API keys + usage plans** (REST) for partner throttling

**🔹 Throttling:** account-level default plus per-stage/per-method limits and **usage plans**; clients get `429` when exceeded.

**🔹 Integration types:** Lambda proxy, HTTP proxy, AWS service, VPC Link (private ALB/NLB), mock.

**🔹 Common issues**
- `504` — integration exceeded the timeout (29 seconds by default)
- `403` — missing authorizer permission / WAF block / resource policy
- CORS errors — enable CORS on the API and return the correct headers
- Use **stages**, **canary deployments**, and CloudWatch **access logs** for operations

<br>

---

### Q30. How does CloudFront work? Scenario: users see stale content after a deployment — how do you fix it?

#### 📌 Short Explanation
Tests CDN, caching strategy, and secure origin access.

#### ✅ Answer
**CloudFront** is a global CDN that caches content at edge locations close to users. For stale content, **invalidate** the cache or, better, use **versioned file names** and appropriate cache headers/TTLs.

#### 📖 Detailed Explanation

**🔹 How it works**
```text
User → nearest Edge Location → (cache hit? return) → (miss) → Origin (S3 / ALB / API) → cache & return
```

**🔹 Key concepts**
- **Distribution**, **origins** (S3, ALB, custom), **behaviors** (path-based rules), **cache policy**, **TTL**
- **OAC (Origin Access Control)** keeps the S3 bucket private so only CloudFront can read it (replaces legacy OAI)
- **HTTPS** with ACM certificates, **WAF**, geo restriction, signed URLs/cookies
- **Lambda@Edge / CloudFront Functions** for edge logic

**🔹 Fixing stale content**
```bash
aws cloudfront create-invalidation --distribution-id E123ABC --paths "/index.html" "/static/*"
```
- Prefer **hashed filenames** (`app.8f3a1.js`) with long TTLs, and a short TTL (or `no-cache`) for `index.html`
- First 1,000 invalidation paths per month are free, then charged

**🔹 Other common issues**

| Symptom | Cause |
|---------|-------|
| 403 from S3 origin | OAC/bucket policy missing |
| 502/504 | Origin unreachable, TLS mismatch, timeout |
| Low cache hit ratio | Forwarding unneeded headers/cookies/query strings |

<br>

---

### Q31. What is AWS Certificate Manager (ACM)? Scenario: your certificate isn't available for CloudFront or fails to validate — what's wrong?

#### 📌 Short Explanation
Tests TLS certificate management and region-specific gotchas.

#### ✅ Answer
**ACM** provisions, manages, and automatically renews **public SSL/TLS certificates** (free) for integrated AWS services such as ALB, CloudFront, and API Gateway.

#### 📖 Detailed Explanation

**🔹 Key facts**
- Public certificates are **free**, and renew automatically (when in use and validation records remain)
- Certificates are **regional**; for **CloudFront**, the certificate **must be in `us-east-1`**
- Cannot be installed directly on EC2 (you can't export the private key of public certs); use ALB/CloudFront or **ACM Private CA / imported certs**
- **Imported certificates are not auto-renewed**

**🔹 Validation methods**

| Method | Notes |
|--------|-------|
| **DNS validation** (recommended) | Add a CNAME in Route 53; supports auto-renewal |
| **Email validation** | Sent to domain contacts; manual |

**🔹 Troubleshooting**
- Stuck in `Pending validation` → CNAME not added, wrong hosted zone, or DNS propagation delay
- Certificate not shown for CloudFront → created in the **wrong region**
- Certificate doesn't cover the hostname → use **SAN** or **wildcard** (`*.example.com`)
- Renewal failures → DNS record deleted, or certificate not in use

```bash
aws acm request-certificate --domain-name "*.example.com" --validation-method DNS --region us-east-1
```

<br>

---

### Q32. Explain Amazon CloudWatch. How do you monitor memory usage on EC2 and set alarms?

#### 📌 Short Explanation
CloudWatch is the core AWS monitoring service; this tests metrics, logs, and alarms.

#### ✅ Answer
**CloudWatch** collects **metrics, logs, and events**, supports **alarms** and dashboards. EC2 does **not** publish **memory or disk usage** by default, so install the **CloudWatch Agent**.

#### 📖 Detailed Explanation

**🔹 Components**

| Component | Purpose |
|-----------|---------|
| **Metrics** | Time-series data (CPU, network, custom) |
| **Logs** | Log groups/streams, retention, **Logs Insights** queries |
| **Alarms** | `OK`, `ALARM`, `INSUFFICIENT_DATA` states with actions (SNS, ASG, EC2) |
| **Dashboards** | Visualization |
| **Metric filters** | Turn log patterns into metrics |
| **Container Insights / Application Signals** | ECS/EKS and APM insights |

**🔹 Default EC2 metrics:** CPU, network, disk I/O, status checks — at **5-minute** granularity (1-minute with detailed monitoring).

**🔹 Memory and disk setup**
1. Attach an IAM role with `CloudWatchAgentServerPolicy`
2. Install and configure the **CloudWatch agent** (`mem_used_percent`, `disk_used_percent`)
3. Create an alarm on the custom metric

```bash
aws cloudwatch put-metric-alarm --alarm-name high-mem \
  --metric-name mem_used_percent --namespace CWAgent \
  --statistic Average --period 300 --threshold 80 \
  --comparison-operator GreaterThanThreshold --evaluation-periods 2 \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:ops-alerts
```

**🔹 Logs Insights example**
```text
fields @timestamp, @message | filter @message like /ERROR/ | sort @timestamp desc | limit 20
```

> 💡 Set **log retention** — the default is "never expire", which silently increases cost.

<br>

---

### Q33. What is the difference between CloudWatch, CloudTrail, and AWS Config?

#### 📌 Short Explanation
A very common comparison. Each service answers a different question.

#### ✅ Answer
- **CloudWatch** — *"How is it performing?"* (metrics, logs, alarms)
- **CloudTrail** — *"Who did what, when?"* (API audit trail)
- **AWS Config** — *"What does it look like, and is it compliant?"* (configuration history and rules)

#### 📖 Detailed Explanation

| Service | Purpose | Example question |
|---------|---------|------------------|
| **CloudWatch** | Monitoring and observability | "Why is CPU at 95%?" |
| **CloudTrail** | Governance and audit of API calls | "Who deleted the security group?" |
| **AWS Config** | Resource configuration tracking and compliance | "Was this S3 bucket ever public? When did it change?" |

**🔹 Example incident:** a security group was opened to `0.0.0.0/0`.
- **CloudTrail** → who made the `AuthorizeSecurityGroupIngress` call and from which IP
- **Config** → previous vs current configuration and compliance status
- **CloudWatch/EventBridge** → real-time alert when it happens

> **Key takeaway:** Performance → CloudWatch · Audit → CloudTrail · Compliance/config drift → Config.

<br>

---

### Q34. Explain AWS CloudTrail. Scenario: someone deleted a resource — how do you find out who did it?

#### 📌 Short Explanation
Tests audit and incident investigation skills.

#### ✅ Answer
Search **CloudTrail Event history** (or Athena over trail logs in S3) for the delete event, and identify the **user/role, source IP, time, and request parameters**.

#### 📖 Detailed Explanation

**🔹 Key facts**
- **Management events** are logged by default; **Event history** keeps **90 days**
- Create a **trail** to deliver logs to **S3** for long-term retention (and optionally CloudWatch Logs)
- **Data events** (S3 object-level, Lambda invoke) must be enabled explicitly (extra cost)
- Use an **organization trail** and **log file validation** for integrity
- Protect the log bucket (separate account, restricted access, MFA delete/Object Lock)

**🔹 Investigation**
```bash
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=DeleteSecurityGroup \
  --max-results 5
```
Review `userIdentity`, `sourceIPAddress`, `eventTime`, and `requestParameters`. For long-term analysis, query logs with **Athena**.

**🔹 Prevention and alerting**
- **EventBridge rule / CloudWatch metric filter** on critical APIs (root login, `DeleteTrail`, IAM policy changes)
- Tag resources, use **SCPs** to block dangerous actions, require MFA for sensitive operations

<br>

---

### Q35. What is AWS Config? Scenario: how do you detect and automatically remediate non-compliant resources?

#### 📌 Short Explanation
Tests governance and compliance automation.

#### ✅ Answer
**AWS Config** continuously records resource configurations and evaluates them against **Config Rules**. Non-compliant resources can be fixed automatically using **remediation actions** (SSM Automation documents).

#### 📖 Detailed Explanation

**🔹 Capabilities**
- Configuration **timeline/history** and relationships
- **Managed rules** (e.g., `s3-bucket-public-read-prohibited`, `encrypted-volumes`, `restricted-ssh`) and **custom rules** (Lambda)
- **Conformance packs** (CIS, PCI) and **multi-account aggregators**

**🔹 Scenario: S3 bucket made public**
1. Enable Config and the rule `s3-bucket-public-read-prohibited`
2. Resource becomes **NON_COMPLIANT**
3. **Auto-remediation** runs the SSM document `AWS-DisableS3BucketPublicReadWrite`
4. Notification via EventBridge → SNS

```bash
aws configservice put-config-rule --config-rule file://s3-public-rule.json
aws configservice describe-compliance-by-config-rule
```

**🔹 Config vs CloudTrail:** Config shows **state over time**; CloudTrail shows the **API actions** that caused changes.

<br>

---

### Q36. What is Amazon Inspector? How is it different from GuardDuty?

#### 📌 Short Explanation
Tests knowledge of the AWS security services landscape.

#### ✅ Answer
**Amazon Inspector** scans workloads for **software vulnerabilities (CVEs)** and unintended network exposure. **GuardDuty** detects **active threats and suspicious behavior**.

#### 📖 Detailed Explanation

| Aspect | Amazon Inspector | Amazon GuardDuty |
|--------|------------------|------------------|
| **Focus** | Vulnerability management | Threat detection |
| **Scans** | EC2 (via SSM Agent), **ECR images**, Lambda functions | CloudTrail, VPC Flow Logs, DNS logs, EKS audit logs |
| **Finds** | CVEs, vulnerable packages, network reachability | Crypto-mining, credential exfiltration, unusual API calls |
| **Nature** | Proactive (before attack) | Reactive (detect ongoing attack) |

**🔹 Inspector highlights**
- **Continuous**, automated scanning (no manual scan scheduling)
- Findings have **severity and a risk score**, and integrate with **Security Hub** and **EventBridge**
- Common use: block CI/CD deployments or alert when ECR images contain **critical CVEs**

**🔹 Typical remediation flow:** Inspector finding → EventBridge → ticket/SSM Patch Manager run → re-scan.

<br>

---

### Q37. Explain AWS KMS. What is envelope encryption? Scenario: you get `AccessDenied` when decrypting an object.

#### 📌 Short Explanation
Tests encryption fundamentals and key access control.

#### ✅ Answer
**KMS** manages cryptographic keys and integrates with most AWS services. **Envelope encryption** uses a KMS key to protect a **data key**, which encrypts the actual data. Decrypt failures are usually **key policy/IAM** issues.

#### 📖 Detailed Explanation

**🔹 Key types**

| Type | Control |
|------|---------|
| **AWS owned keys** | Fully managed by AWS, invisible |
| **AWS managed keys** (`aws/s3`) | Managed by AWS, visible in your account, can't change policy |
| **Customer managed keys (CMK)** | You control policy, rotation, and deletion |

**🔹 Envelope encryption**
```text
KMS key → GenerateDataKey → plaintext data key (encrypt data) + encrypted data key (stored with data)
Decrypt: KMS decrypts the data key → use it to decrypt data
```
KMS encrypts up to 4 KB directly; larger data uses envelope encryption.

**🔹 Access control:** **Key policy is the primary authority** (IAM alone isn't enough unless the key policy allows IAM). Grants provide temporary access.

**🔹 `AccessDenied` on decrypt — checklist**
- Does the **key policy** allow the principal (or enable IAM policies)?
- Does the IAM role have `kms:Decrypt` / `kms:GenerateDataKey`?
- **Cross-account:** both the key policy and the caller's IAM policy must allow it
- Is the **key disabled or scheduled for deletion** (7–30 day waiting period)?
- Are **VPC endpoint** or encryption-context conditions blocking?

**🔹 Rotation:** CMK automatic rotation can be enabled (old key material is retained to decrypt old data).

<br>

---

### Q38. What is Amazon EventBridge? Give scenarios for scheduling and reacting to AWS events.

#### 📌 Short Explanation
Tests event-driven architecture knowledge.

#### ✅ Answer
**EventBridge** is a serverless event bus that routes events from AWS services, SaaS apps, and your own apps to targets (Lambda, SQS, SNS, Step Functions, etc.) using **rules**.

#### 📖 Detailed Explanation

**🔹 Core concepts:** event bus (default, custom, partner), **event pattern** rules, schedule rules/EventBridge Scheduler, targets, archive and replay, cross-account/cross-region buses, DLQs and retries.

**🔹 Scenario 1: react to an EC2 state change**
```json
{
  "source": ["aws.ec2"],
  "detail-type": ["EC2 Instance State-change Notification"],
  "detail": { "state": ["stopped", "terminated"] }
}
```
Target: Lambda or SNS to alert/log.

**🔹 Scenario 2: schedule a nightly job**
```text
cron(0 2 * * ? *)  →  Lambda that stops dev instances
```

**🔹 Scenario 3: automatic security response**
GuardDuty/Config finding → EventBridge → Lambda/SSM remediates (isolate instance, revoke key).

**🔹 EventBridge vs SNS vs SQS**

| Service | Model | Best for |
|---------|-------|----------|
| **EventBridge** | Event bus with filtering and routing | Event-driven integration, AWS events, SaaS events |
| **SNS** | Pub/sub fan-out | Push notifications to many subscribers |
| **SQS** | Message queue (pull) | Decoupling and buffering work |

<br>

---

### Q39. What is the difference between AWS Secrets Manager and SSM Parameter Store?

#### 📌 Short Explanation
Both store configuration and secrets. Interviewers want to know when to use which.

#### ✅ Answer
**Secrets Manager** is designed for secrets with **built-in automatic rotation**. **Parameter Store** is a simpler, cheaper configuration store that can also hold encrypted values (`SecureString`).

#### 📖 Detailed Explanation

| Feature | Secrets Manager | SSM Parameter Store |
|---------|-----------------|---------------------|
| **Primary use** | Secrets (DB credentials, API keys) | Configuration and simple secrets |
| **Automatic rotation** | ✅ Built-in (Lambda), native for RDS | ❌ Not native (DIY) |
| **Cost** | Per secret per month + API calls | Standard tier free, Advanced tier charged |
| **Max size** | 64 KB | 4 KB (Standard) / 8 KB (Advanced) |
| **Encryption** | Always KMS | `SecureString` with KMS (optional) |
| **Cross-region replication** | ✅ Yes | ❌ No |
| **Versioning** | ✅ Staging labels (`AWSCURRENT`) | ✅ Versions |

**🔹 Reading a secret**
```bash
aws secretsmanager get-secret-value --secret-id prod/db/creds --query SecretString --output text
aws ssm get-parameter --name /prod/app/api-url --with-decryption
```

**🔹 Rotation flow (RDS):** Secrets Manager triggers a Lambda that creates a new password, updates the DB, tests it, and promotes the new version — apps that fetch the secret at runtime pick it up automatically.

**🔹 Best practices**
- Never hardcode secrets or store them in Git/AMIs/env files
- Grant access via **IAM roles** (least privilege) and audit via CloudTrail
- Rule of thumb: **rotating credentials → Secrets Manager; plain config → Parameter Store**

<br>

---

### Q40. What is AWS Systems Manager Session Manager? Why is it better than SSH/bastion hosts, and how do you troubleshoot it?

#### 📌 Short Explanation
Tests modern, secure instance-access practices.

#### ✅ Answer
**Session Manager** provides secure shell access to instances through the **SSM agent** — with **no inbound ports, no SSH keys, and no bastion host**, using IAM for authorization and full audit logging.

#### 📖 Detailed Explanation

**🔹 SSH/Bastion vs Session Manager**

| Aspect | SSH / Bastion | Session Manager |
|--------|---------------|-----------------|
| **Inbound port 22** | Required | ❌ Not needed |
| **Key management** | PEM files to distribute/rotate | None (IAM-based) |
| **Auditing** | Limited | Logged to CloudTrail, S3, CloudWatch Logs |
| **Access control** | Network + keys | IAM policies, MFA, tags |
| **Bastion cost** | Extra instance | None |

**🔹 Requirements**
1. **SSM Agent** installed (preinstalled on Amazon Linux, Ubuntu AMIs)
2. Instance profile with `AmazonSSMManagedInstanceCore`
3. Network path to `ssm`, `ssmmessages`, `ec2messages` endpoints (internet/NAT or **VPC endpoints**)

```bash
aws ssm start-session --target i-0123456789abcdef0
aws ssm start-session --target i-0123 --document-name AWS-StartPortForwardingSession \
  --parameters "portNumber=3306,localPortNumber=3307"
```

**🔹 Instance not showing in Fleet Manager? Check**
- Missing/incorrect IAM role · agent stopped (`systemctl status amazon-ssm-agent`)
- No outbound HTTPS/VPC endpoints · wrong region · very old agent
- User lacks `ssm:StartSession` permission

**🔹 Related capabilities:** Run Command, Patch Manager, State Manager, Parameter Store, Automation.

<br>