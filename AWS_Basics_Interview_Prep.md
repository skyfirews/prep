# AWS Basics – Interview Notes

> **Target**: Senior Engineers, Technical Architects | **Level**: Amazon, Flipkart, Razorpay

---

## 1. Overview

**What it is**: AWS (Amazon Web Services) is an on-demand, scalable cloud platform providing compute, storage, databases, networking, and hundreds of managed services. You pay for what you use.

**Why companies use it**: Rapid provisioning, global scale, managed services (no server patching), elasticity for traffic spikes, and strong SLAs. Used for e-commerce, APIs, data lakes, ML pipelines.

**When NOT to use it**: Lock-in concerns for highly portable workloads; regulatory requirements for on-prem; when a simpler PaaS (e.g. Vercel, Heroku) meets needs; cost-sensitive small projects without commitment.

---

## 2. Core Concepts (From PDF)

### Compute

| Concept | Definition | Simple Explanation | What Interviewers Test | Common Follow-up |
|---------|------------|--------------------|-------------------------|------------------|
| EC2 | Virtual servers in the cloud | Rent VMs by size/OS; full control | When to use EC2 vs Lambda vs ECS | EC2 vs Lambda trade-offs |
| Auto Scaling | Scale EC2 capacity automatically | Add/remove instances by load or schedule | Cost vs availability | How Auto Scaling works with ALB |
| Lambda | Serverless compute | Run code on events; no servers to manage | Event-driven, cold start | Lambda limits and use cases |
| ECS | Container orchestration | Run Docker containers on AWS | ECS vs EKS | When to use Fargate vs EC2 launch type |
| EKS | Managed Kubernetes | K8s control plane managed by AWS | EKS vs ECS | Multi-tenant vs single-tenant |
| Lightsail | Simplified compute | Fixed-price small VMs | When to use vs EC2 | Limits and migration path |

### Storage

| Concept | Definition | Simple Explanation | What Interviewers Test | Common Follow-up |
|---------|------------|--------------------|-------------------------|------------------|
| S3 | Object storage | Store blobs by key; 11 9's durability | S3 vs EBS vs EFS | S3 consistency model |
| EBS | Block storage | Disks attached to EC2 | Persistence, IOPS | EBS vs instance store |
| EFS | File storage | Shared NFS-style filesystem | Multi-EC2 shared storage | EFS vs EBS |
| Glacier | Archive storage | Low-cost long-term retention | Retrieval tiers and cost | When to use Glacier |

### Database

| Concept | Definition | Simple Explanation | What Interviewers Test | Common Follow-up |
|---------|------------|--------------------|-------------------------|------------------|
| RDS | Managed SQL | MySQL, PostgreSQL, etc.; backups, Multi-AZ | RDS vs DynamoDB | Multi-AZ failover flow |
| Aurora | AWS-native SQL | MySQL/PostgreSQL compatible; high performance | Aurora vs RDS | Aurora scaling and cost |
| DynamoDB | NoSQL serverless | Key-value/document; auto scale | When to use DynamoDB | Partition key design |
| ElastiCache | In-memory cache | Redis/Memcached | Cache strategy | ElastiCache vs Redis self-hosted |
| Redshift | Data warehouse | Analytics, columnar | Redshift vs RDS | When to use Redshift |

### Networking & CDN

| Concept | Definition | Simple Explanation | What Interviewers Test | Common Follow-up |
|---------|------------|--------------------|-------------------------|------------------|
| VPC | Virtual private network | Isolated network in AWS | Public vs private subnet | Default VPC and custom VPC |
| Subnet | Segment of VPC IP range | Public (IGW) vs private (NAT) | Why private subnets for app servers | CIDR and subnet sizing |
| Internet Gateway | VPC attachment for internet | Enables public access | IGW vs NAT Gateway | One IGW per VPC |
| NAT Gateway | Outbound internet for private subnet | Private instances can reach internet | Cost and HA | NAT Gateway vs NAT instance |
| Security Groups | Instance-level firewall | Stateful; allow/deny by port and source | SG vs NACL | Default SG rules |
| NACL | Subnet-level firewall | Stateless; rule order matters | When to use NACL | NACL vs Security Group |
| Load Balancer (ALB/NLB) | Distribute traffic | ALB: L7; NLB: L4, low latency | ALB vs NLB | Sticky sessions, target groups |
| CloudFront | CDN | Edge caching, DDoS mitigation | Cache behavior, TTL | CloudFront vs S3 direct |

### Security & Identity

| Concept | Definition | Simple Explanation | What Interviewers Test | Common Follow-up |
|---------|------------|--------------------|-------------------------|------------------|
| IAM User | Human identity | Long-term credentials for people | IAM User vs Role | When to use roles |
| IAM Role | Service/identity | Temporary credentials; no long-term keys | Why roles over users for EC2 | AssumeRole, trust policy |
| IAM Policy | Permissions document | JSON allow/deny on resources/actions | Least privilege | Inline vs managed policy |
| MFA | Multi-factor auth | Extra factor for console/CLI | Where to enforce MFA | MFA for root |
| KMS | Key management | Create/use encryption keys | KMS vs Secrets Manager | Key rotation |
| Secrets Manager | Store secrets | DB passwords, API keys; rotation | When to use vs Parameter Store | Cost and rotation |

### DevOps & IaC

| Concept | Definition | Simple Explanation | What Interviewers Test | Common Follow-up |
|---------|------------|--------------------|-------------------------|------------------|
| CodePipeline | CI/CD orchestration | Source → Build → Deploy | Pipeline stages | Approval actions |
| CloudFormation | IaC (AWS-native) | Templates for resources | CFN vs Terraform | Drift detection |
| Terraform | IaC (multi-cloud) | HCL; state; provider AWS | When to use Terraform | State and locking |

### HA & DR

| Concept | Definition | Simple Explanation | What Interviewers Test | Common Follow-up |
|---------|------------|--------------------|-------------------------|------------------|
| Multi-AZ | Multiple AZs in one region | HA within region; sync replication | Multi-AZ vs Multi-Region | RDS Multi-AZ failover |
| Multi-Region | Multiple regions | DR, latency; async replication | When to go multi-region | Data consistency |
| Failover | Switch to standby on failure | Automated or manual | RDS/Route53 failover | RTO/RPO |

---

## 3. How It Works (Internals)

**Request flow (high level)**  
Client → Route 53 (DNS) → CloudFront (if CDN) → ALB → Security Group → EC2/ECS/Lambda.  
VPC provides isolation; subnets segment IP space; route tables define where traffic goes (IGW, NAT, peering).

**Data flow (storage)**  
S3: object key → partition (prefix + hash) → bucket in a region; strong read-after-write for new objects.  
EBS: block device attached to one AZ; snapshots stored in S3.  
RDS: primary in one AZ, standby in another (Multi-AZ); failover via DNS flip.

**IAM evaluation**  
Request comes with principal (user/role) + action + resource. IAM evaluates all attached policies; explicit Deny wins; no match = Deny.

**Word diagram – Public vs private subnet**  
```
Internet → IGW → [Public Subnet: ALB, NAT GW] → [Private Subnet: App EC2, RDS]
                    ↑                                    ↑
              Route 0.0.0.0/0 → IGW              Route 0.0.0.0/0 → NAT GW
```

---

## 4. Real-World Use Case

**E-commerce backend**

- **Where AWS fits**: Web/app traffic (ALB + EC2/ECS), static assets (S3 + CloudFront), catalog and orders (RDS or Aurora), session/cart cache (ElastiCache), search (OpenSearch), queues (SQS), events (EventBridge).
- **Why this approach**: Managed DB and cache reduce ops; Auto Scaling handles Black Friday; Multi-AZ and backups meet availability and DR; IAM roles for apps avoid storing DB passwords.
- **Trade-offs**: Vendor lock-in vs speed; RDS cost vs self-managed DB; Lambda cold start vs always-on EC2/ECS for latency-sensitive paths.

---

## 5. Code Example (Minimal but Powerful)

```python
# boto3 – list S3 buckets (Python SDK)
import boto3

# Uses default credential chain: env vars, ~/.aws/credentials, IAM role
client = boto3.client("s3")
response = client.list_buckets()

for bucket in response["Buckets"]:
    print(bucket["Name"])
# Don't hardcode keys; use IAM role on EC2/ECS/Lambda.
# Wrong: storing access key in code → security risk, rotation nightmare.
```

```python
# IAM role preferred over access keys
# EC2 instance with role "MyAppRole" – no keys in code or config
s3 = boto3.resource("s3")
obj = s3.Object("my-bucket", "path/key.txt")
body = obj.get()["Body"].read()  # Time: O(size), Space: O(size); use streaming for large files
```

---

## 6. Best Practices (Interview-Grade)

| Do | Don't |
|----|--------|
| Use IAM roles for EC2/Lambda/ECS | Don't put access keys in code or repo |
| Put app/DB in private subnets | Don't put DB in public subnet |
| Enable MFA for root and human users | Don't use root for daily work |
| Use least-privilege IAM policies | Don't use * on resources when not needed |
| Multi-AZ for production DBs | Don't rely on single AZ for HA |
| Tag resources for cost and ops | Don't leave untagged prod resources |
| Use CloudFormation/Terraform for infra | Don't create prod manually and forget IaC |

**Security**: Encrypt sensitive data (S3, EBS, RDS); use Secrets Manager or Parameter Store; enable CloudTrail and Config.  
**Scalability**: Design for horizontal scaling (stateless app, ALB, Auto Scaling); use managed services to avoid single points of failure.

---

## 7. Common Mistakes (Interview Red Flags)

| Mistake | Why wrong | Correct mental model |
|---------|-----------|------------------------|
| "We use EC2 for everything" | Lambda/ECS can be cheaper and simpler for event-driven or container workloads | Choose compute by workload: Lambda (event, short), ECS (containers), EC2 (full control) |
| "Security Groups and NACL are the same" | SG is stateful (return traffic allowed); NACL is stateless and order-dependent | SG = instance firewall; NACL = subnet firewall; use both when you need subnet-level rules |
| "Public subnet is for public-facing, private for backend" | Public subnet = has route to IGW; private = no IGW, use NAT for outbound | Put only ALB/NAT/Bastion in public; app and DB in private |
| "Multi-AZ and Multi-Region are the same" | Multi-AZ = one region, sync; Multi-Region = DR, async, higher latency | Multi-AZ for HA in one region; Multi-Region for DR and geo |
| "IAM users for applications" | Long-lived keys are hard to rotate and often leak | Use IAM roles (EC2 role, Lambda execution role) for apps |

---

## 8. Interview Questions & Answers

### A. Basic (Warm-up)

**Q: What is the difference between IAM User and IAM Role?**  
**Short answer**: User is for humans (long-term credentials). Role is for services or assumed identity (temporary credentials).  
**Follow-up trap**: "When would you give an EC2 instance an IAM user?"  
**Ideal answer**: Never. Use an IAM role attached to the EC2 instance; the instance gets temporary credentials automatically. No keys to store or rotate.

**Q: What is the difference between S3 and EBS?**  
**Short answer**: S3 is object storage (key-value, HTTP API, unlimited size). EBS is block storage (disk attached to one EC2, low-latency block I/O).  
**Follow-up**: "Can EBS be shared by multiple EC2 instances?"  
**Ideal answer**: Not directly. For shared filesystem use EFS. Some workloads use shared storage solutions on top of EBS (e.g. clustered DB) with careful design.

### B. Intermediate (Most common)

**Q: Explain Security Groups vs NACL.**  
**Short answer**: Security Groups are stateful, applied at ENI/instance level; you allow inbound, return traffic is automatically allowed. NACL is stateless, subnet-level; you must allow both inbound and outbound; rules are evaluated in order.  
**Follow-up**: "If I allow port 80 in NACL but not in Security Group, does the request reach the instance?"  
**Ideal answer**: No. Both NACL and Security Group must allow the traffic. NACL is first (subnet), then Security Group (instance).

**Q: When do you use EC2 vs Lambda?**  
**Short answer**: EC2 when you need long-running processes, specific OS/lib, or persistent state. Lambda for event-driven, short-running (minutes), variable load, and no server management.  
**Follow-up**: "Our Lambda runs for 20 minutes and reads a 2 GB file from S3. Is that fine?"  
**Ideal answer**: No. Lambda has a 15-minute max timeout and limited memory. Use EC2, ECS, or Step Functions + batch for long/heavy jobs.

### C. Advanced / Tricky (Senior-level)

**Q: How does RDS Multi-AZ failover work?**  
**Short answer**: Primary replicates synchronously to standby in another AZ. On primary failure, AWS flips DNS to the standby (same endpoint); storage is already replicated.  
**Follow-up**: "Do we need to change the connection string after failover?"  
**Ideal answer**: No. The RDS endpoint (CNAME) is updated to point to the new primary. App reconnects to the same hostname. Brief downtime during failover (typically 1–2 minutes).

**Q: Design a highly available web app on AWS in one region.**  
**Ideal answer**: Two or more AZs; ALB in multiple AZs; Auto Scaling group with EC2/ECS across AZs; RDS/Aurora Multi-AZ; ElastiCache with replication; static assets on S3 + CloudFront; Route 53 for DNS. No single point of failure in one region.

---

## 9. Quick Revision

- IAM: Prefer **roles** for services; **users** for humans; **least privilege**.
- **EC2** = VMs; **Lambda** = serverless, event-driven; **ECS** = Docker on AWS.
- **S3** = object; **EBS** = block (one EC2); **EFS** = shared file.
- **RDS** = managed SQL, Multi-AZ; **DynamoDB** = serverless NoSQL.
- **Security Group** = stateful, instance; **NACL** = stateless, subnet.
- **Public subnet** = route to IGW; **private subnet** = outbound via NAT.
- **ALB** = L7, path/host routing; **NLB** = L4, low latency.
- **Multi-AZ** = HA in one region; **Multi-Region** = DR / global.

---

## 10. References

- [AWS Documentation](https://docs.aws.amazon.com/)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
