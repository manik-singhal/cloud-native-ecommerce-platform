# Architecture Tradeoffs

This document explains the key architectural and cost decisions made in this project.

The goal wasn't to optimize for every scenario. It was to make reasonable, defensible choices that reflect how platform teams think in production systems.

---

## Why EKS Instead of EC2 + Docker

Running containers on EC2 with Docker is simpler upfront, but it shifts more operational responsibility onto you. Container restarts, service discovery, and traffic routing all need to be handled manually or with additional tooling.

Kubernetes provides these natively:
- Automatic restarts when containers fail
- Built-in service discovery
- Declarative state management
- Standardized deployment patterns

For a microservices system, Kubernetes provides a clearer operational model even though the initial setup is more involved.

---

## Why Not Self-Managed Kubernetes

Self-managing Kubernetes means maintaining the control plane, handling upgrades, and ensuring cluster stability yourself.

For this project, I wanted to focus on:
- Application deployment
- Service communication
- CI/CD and GitOps workflows

Using EKS removes the need to manage the Kubernetes control plane and reflects how many teams actually run Kubernetes in production. This let me focus on using the platform rather than maintaining it.

---

## Why Infrastructure Isn't Always Running

EKS clusters cost money even when idle. Since this project demonstrates architecture and deployment workflows, infrastructure is created on demand and destroyed afterward using Terraform.

This approach:
- Reduces unnecessary cloud spend
- Encourages intentional infrastructure lifecycle management
- Reflects cost-aware engineering

For interviews and demonstrations, architecture diagrams and documentation replace a permanently running system.

---

## How I'd Scale This

If usage increased significantly, I'd rely on Kubernetes-native scaling mechanisms:
- Increase replicas for stateless services
- Adjust resource requests and limits based on actual usage
- Ensure services are horizontally scalable

These changes would be incremental and guided by observed behavior, not premature optimization.

---

## Budget Tradeoffs

If cost constraints were tighter, I'd prioritize simplicity:
- Run smaller clusters
- Reduce non-essential components
- Lower replica counts where acceptable

The goal would be balancing reliability with cost based on actual usage, not assumptions.

---

## Summary

The decisions in this project prioritize:
- Operational clarity
- Cost awareness
- Production-aligned patterns

The architecture is designed to be understandable, maintainable, and adaptable as needs change.
