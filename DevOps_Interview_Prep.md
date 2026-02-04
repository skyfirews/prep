# DevOps – Interview Notes

> **Target**: Senior Engineers, SRE, Platform | **Level**: Amazon, Flipkart, Razorpay

---

## 1. Overview

**What it is**: DevOps is a culture and set of practices that combine development and operations: automation, CI/CD, IaC, monitoring, and collaboration to deliver software faster and more reliably.

**Why companies use it**: Faster releases, fewer manual errors, repeatable environments (IaC), quick rollback, better observability, and alignment between dev and ops (SRE).

**When NOT to use it**: Very small teams with no ops; legacy systems with no automation path; when compliance forbids certain tooling; over-automation where manual is cheaper and safe.

---

## 2. Core Concepts (From PDF)

### Culture & Practices

| Concept | Definition | Simple Explanation | What Interviewers Test | Common Follow-up |
|---------|------------|--------------------|-------------------------|------------------|
| DevOps | Culture + automation (CI/CD, IaC) | Dev and Ops work together; automate everything | Why DevOps | DevOps vs SRE |
| CI | Continuous Integration | Merge code often; build and test on every commit | Pipeline stages | CI vs CD |
| CD | Continuous Deployment/Delivery | Deploy to prod automatically (or with approval) | Zero downtime | Blue-Green vs Canary |
| IaC | Infrastructure as Code | Define infra in code (Terraform, CloudFormation) | Repeatability | Terraform vs CFN |

### Version Control & CI Tools

| Concept | Definition | Simple Explanation | What Interviewers Test | Common Follow-up |
|---------|------------|--------------------|-------------------------|------------------|
| Git | Version control | Branch, merge, PR, tags | Branching strategy | Merge vs Rebase |
| Jenkins | CI server | Pipelines, agents, jobs | Pipeline design | Jenkins vs GitHub Actions |
| GitHub Actions | CI/CD in GitHub | Workflows, YAML | When to use | Self-hosted vs GitHub-hosted |
| GitLab CI | Integrated CI/CD | .gitlab-ci.yml | Stages, jobs | |

### Deployment Strategies

| Concept | Definition | Simple Explanation | What Interviewers Test | Common Follow-up |
|---------|------------|--------------------|-------------------------|------------------|
| Blue-Green | Two identical envs; switch traffic | Zero downtime; instant rollback | Pros and cons | Cost (2x env) |
| Canary | New version to small % of traffic | Gradual rollout; detect issues early | Risk vs Blue-Green | How to route |
| Rolling | Replace instances one by one | No second full env | Downtime risk | When to use |
| Rollback | Revert to previous version | On failure or incident | How to rollback | Automated vs manual |

### Containerization & Orchestration

| Concept | Definition | Simple Explanation | What Interviewers Test | Common Follow-up |
|---------|------------|--------------------|-------------------------|------------------|
| Docker | Container runtime | Image = snapshot; container = running instance | Docker vs VM | Image layers |
| Dockerfile | Build instructions for image | FROM, RUN, COPY, CMD | Multi-stage build | |
| Kubernetes (K8s) | Container orchestration | Pods, Deployments, Services | K8s vs ECS | When to use K8s |
| Pod | Smallest deployable unit in K8s | One or more containers | Why pods | |
| Deployment | Declarative desired state | Replicas, rolling update | Scaling | |
| ConfigMap / Secrets | Externalized config | Non-sensitive vs sensitive | How to use | |

### Monitoring & Logging

| Concept | Definition | Simple Explanation | What Interviewers Test | Common Follow-up |
|---------|------------|--------------------|-------------------------|------------------|
| Monitoring | Metrics and alerts | CPU, memory, latency, errors | SLI, SLO, SLA | Prometheus vs CloudWatch |
| Logging | Centralized logs | ELK, CloudWatch Logs | Log levels, structure | |
| Tracing | Distributed tracing | Request across services | Jaeger, X-Ray | When to use |
| SLI | Service Level Indicator | Measured metric (e.g. latency p99) | What to measure | |
| SLO | Service Level Objective | Target (e.g. 99.9% uptime) | SLO vs SLA | |
| SLA | Service Level Agreement | Contract with customer | Penalties | |

### Security & Reliability

| Concept | Definition | Simple Explanation | What Interviewers Test | Common Follow-up |
|---------|------------|--------------------|-------------------------|------------------|
| DevSecOps | Security in pipeline | Shift-left; SAST, DAST, secrets | How to secure CI/CD | |
| Secrets management | Vault, Secrets Manager | No secrets in code/repo | Where to store | |
| SAST | Static analysis | Scan code for vulnerabilities | When in pipeline | |
| Failover | Switch to standby on failure | High availability | Automated vs manual | |

---

## 3. How It Works (Internals)

**CI/CD pipeline flow**  
Commit → Push → Webhook → CI (build, test, lint) → Artifact (image/jar) → CD (deploy to staging) → Tests → (Approval) → Deploy to prod. Rollback = redeploy previous artifact or switch traffic (Blue-Green).

**Blue-Green**  
Two envs: Blue (current), Green (new). Deploy to Green; test; switch load balancer to Green; Blue becomes standby. Rollback = switch back to Blue.

**Canary**  
Deploy new version; route 5% traffic to it; monitor; increase to 50%, 100% or rollback.

**Docker**  
Dockerfile → docker build → image (layers). docker run → container (writable layer on top of image). Container is isolated (namespaces, cgroups).

**Kubernetes**  
Control plane (API server, scheduler, etcd) + nodes (kubelet, container runtime). You submit Deployment YAML; scheduler places Pods on nodes; kubelet runs containers. Service gives stable network to Pods.

**Word diagram – CI/CD**  
```
Code → [Git] → [CI: Build, Test] → [Artifact] → [CD: Deploy] → [Prod]
                ↓                      ↓
           [Jenkins / Actions]    [Blue-Green / Canary]
```

---

## 4. Real-World Use Case

**Microservices API platform**

- **Where DevOps fits**: Git (mono or multi-repo); CI (build, unit/integration tests, Docker image); CD (deploy to K8s or ECS); IaC (Terraform for VPC, EKS, RDS); monitoring (Prometheus + Grafana); logging (ELK or CloudWatch); secrets (Vault or AWS Secrets Manager).
- **Why this approach**: Repeatable builds and deploys; zero-downtime (Blue-Green or rolling); quick rollback; observability for incidents.
- **Trade-offs**: Complexity vs benefit; cost of tooling and pipelines; need for runbooks and on-call.

---

## 5. Code Example (Minimal but Powerful)

```yaml
# GitHub Actions – CI pipeline (simplified)
name: CI
on: [push]
jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"
      - name: Install deps
        run: pip install -r requirements.txt
      - name: Test
        run: pytest
      - name: Build Docker image
        run: docker build -t myapp:${{ github.sha }} .
# Wrong: no test step → broken code can be deployed.
# Wrong: secrets in env without masking → leak risk.
```

```hcl
# Terraform – minimal EC2 (IaC)
resource "aws_instance" "app" {
  ami           = "ami-xxxx"
  instance_type = "t3.micro"
  subnet_id     = aws_subnet.private.id
  # Prefer IAM role over access keys
}
# Wrong: hardcoding AMI/region → not portable. Use variables.
```

---

## 6. Best Practices (Interview-Grade)

| Do | Don't |
|----|--------|
| Automate build, test, deploy | Don't deploy manually to prod repeatedly |
| Use IaC for all environments | Don't create prod by hand and forget |
| Store secrets in vault/Secrets Manager | Don't put secrets in code or repo |
| Use Blue-Green or Canary for zero downtime | Don't do big-bang deploy without rollback plan |
| Define SLIs/SLOs and alert on them | Don't alert on everything (alert fatigue) |
| Document runbooks and rollback steps | Don't assume only one person knows how to rollback |

**Security**: Shift-left (SAST, dependency scan); scan container images; least privilege in pipelines.  
**Reliability**: Automated rollback; health checks; failover and DR tests.

---

## 7. Common Mistakes (Interview Red Flags)

| Mistake | Why wrong | Correct mental model |
|---------|-----------|------------------------|
| "CI and CD are the same" | CI = build/test on commit; CD = deploy to prod | CI is quality gate; CD is delivery automation |
| "Docker and Kubernetes are the same" | Docker = run containers; K8s = orchestrate many containers | Docker runs one; K8s schedules, scales, heals |
| "Blue-Green and Canary are the same" | Blue-Green = full switch; Canary = gradual % | Use Blue-Green for instant switch; Canary for gradual risk |
| "We don't need rollback" | Every deploy can fail | Always have one-click or scripted rollback |
| "Monitoring and logging are the same" | Metrics = numbers over time; logs = events | Use both; metrics for SLOs, logs for debugging |

---

## 8. Interview Questions & Answers

### A. Basic (Warm-up)

**Q: What is the difference between CI and CD?**  
**Short answer**: CI = continuous integration: build and test on every commit. CD = continuous delivery/deployment: deploy to prod (with or without approval) after CI passes.  
**Follow-up**: "Do we need both?"  
**Ideal answer**: Yes. CI ensures code is buildable and tested; CD automates release. Without CI, CD can deploy broken code. Without CD, releases stay manual and slow.

**Q: What is Blue-Green deployment?**  
**Short answer**: Two identical environments (Blue = current, Green = new). Deploy to Green; test; switch traffic to Green. Rollback = switch back to Blue.  
**Follow-up**: "What is the downside?"  
**Ideal answer**: Cost (two full environments) and need to keep data/config in sync (e.g. DB migrations). Canary reduces cost but adds complexity of traffic splitting.

### B. Intermediate (Most common)

**Q: How do you deploy with zero downtime?**  
**Short answer**: Blue-Green (switch traffic at once) or rolling update (replace instances one by one while load balancer drains). Ensure health checks so LB only sends traffic to healthy instances.  
**Follow-up**: "What if the new version has a bug?"  
**Ideal answer**: Rollback: switch traffic back (Blue-Green) or deploy previous version (rolling). Automated rollback on health check failure or alert reduces impact.

**Q: Docker vs VM?**  
**Short answer**: VM = full OS on hypervisor; heavy, slow boot. Container = process isolation on host kernel; shares OS, fast start, smaller. Docker is for app portability; VM for full isolation or different OS.  
**Follow-up**: "When would you use a VM instead of Docker?"  
**Ideal answer**: When you need a different kernel/OS, strict isolation (e.g. multi-tenant), or legacy app that doesn't containerize well.

### C. Advanced / Tricky (Senior-level)

**Q: How do you secure secrets in CI/CD?**  
**Ideal answer**: Never store secrets in code or repo. Use a secrets manager (Vault, AWS Secrets Manager). In pipeline: fetch secrets at runtime; use short-lived tokens; restrict pipeline identity (least privilege). Mask secrets in logs. Rotate regularly.

**Q: What happens when a deployment fails in production?**  
**Ideal answer**: Detect via health checks or alerts. Rollback immediately (automated or manual): revert to previous artifact or switch traffic (Blue-Green). Post-incident: root cause, fix pipeline or app, add tests or checks to prevent recurrence. Document in runbook.

---

## 9. Quick Revision

- **CI** = build + test on commit; **CD** = deploy to prod.
- **Blue-Green** = two envs, switch traffic; **Canary** = gradual % traffic.
- **Docker** = containers; **K8s** = orchestration (Pods, Deployments, Services).
- **IaC** = Terraform, CloudFormation; **repeatable** infra.
- **SLI** = measured; **SLO** = target; **SLA** = contract.
- **Secrets** in vault/manager; **never** in repo.
- **Always** have rollback plan.

---

## 10. References

- [Martin Fowler – Continuous Integration](https://martinfowler.com/articles/continuousIntegration.html)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Terraform Documentation](https://www.terraform.io/docs)
