# Architecture Tradeoffs & Cost Decisions

This document explains the key architectural and cost-related decisions
made in this project.

The intent was not to optimize for every scenario, but to make reasonable,
defensible choices similar to how platform teams think in real systems.

---

## Why EKS Instead of EC2 + Docker

Running containers directly on EC2 using Docker is simpler initially,
but it places more operational responsibility on the engineer.

With EC2 + Docker, tasks like container restarts, service discovery,
and traffic routing need to be handled manually or through additional tooling.

Kubernetes (via EKS) provides these capabilities natively:
- Automatic restarts of failed containers
- Built-in service discovery
- Declarative desired state
- Standardized deployment patterns

For a microservices-based system, EKS provides a clearer operational model
even if the initial setup is more involved.

---

## Why Not Self-Managed Kubernetes

Self-managing Kubernetes requires maintaining the control plane,
handling upgrades, and ensuring cluster stability.

For this project, the focus was on:
- Application deployment
- Service communication
- CI/CD and GitOps workflows

Using Amazon EKS removes the need to manage the Kubernetes control plane
directly and reflects how many teams run Kubernetes in production today.

This allowed the project to focus on platform usage rather than
platform maintenance.

---

## Why Infrastructure Is Not Kept Running Continuously

Running EKS clusters continuously incurs cost, even when there is no traffic.

Since this project is meant to demonstrate architecture, deployment,
and operational understanding, infrastructure is created on demand
and destroyed afterward using Terraform.

This approach:
- Reduces unnecessary cloud spend
- Encourages intentional infrastructure lifecycle management
- Reflects cost-aware engineering behavior

For demonstrations and interviews, architecture diagrams and documentation
are used instead of a permanently running system.

---

## Scaling Considerations (High-Level)

If usage increased significantly, scaling would primarily rely on
Kubernetes-native mechanisms.

At a high level, this would involve:
- Increasing replicas for stateless services
- Adjusting resource requests and limits
- Ensuring services are horizontally scalable

These changes would be incremental and guided by observed behavior,
rather than premature optimization.

---

## Budget Considerations (High-Level)

If cost constraints were stricter, tradeoffs would favor simplicity.

This could involve:
- Reducing cluster size
- Limiting non-essential components
- Running fewer replicas where acceptable

The core goal would be balancing reliability with cost,
based on actual usage rather than assumptions.

---

## Summary

The decisions in this project prioritize:
- Operational clarity
- Cost awareness
- Production-aligned patterns

Rather than optimizing for extreme scenarios, the architecture is designed
to be understandable, maintainable, and adaptable as requirements evolve.
