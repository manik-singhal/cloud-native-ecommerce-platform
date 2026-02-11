# Cloud-Native E-Commerce Platform (AWS EKS + GitOps)

This project demonstrates a production-style, cloud-native microservices
e-commerce platform deployed on AWS using Kubernetes (EKS), Terraform,
CI/CD, and GitOps principles.

The focus of this project is not just deployment, but **architecture decisions,
failure handling, automation, and cost control** — similar to how real
platform and SRE teams operate in production environments.

---

## Architecture Overview

The application is composed of multiple independently deployable
microservices running on Kubernetes (Amazon EKS).

Infrastructure such as VPC, EKS cluster, networking, and IAM roles is
provisioned using Terraform to ensure repeatability and version control.

Continuous Integration (CI) is handled using GitHub Actions, while
Continuous Deployment (CD) follows GitOps principles using Argo CD,
where Git acts as the single source of truth for the cluster state.

```mermaid
graph TD
    subgraph "Public Internet"
        User((User)) -->|HTTPS| ALB[AWS Application Load Balancer]
    end

    subgraph "AWS EKS Cluster (ap-south-1)"
        ALB -->|Ingress| Proxy[frontend-proxy]
        Proxy -->|gRPC/HTTP| Services{Microservices}

        subgraph "src/ Microservices"
            Services --> Product[product-catalog]
            Services --> Cart[cart]
            Services --> Checkout[checkout]
            Cart --> Valkey[(Valkey Cache)]
        end

        subgraph "Observability"
            Services -.->|OTLP| OTEL[OTEL Collector]
            OTEL --> Jaeger[Jaeger Traces]
            OTEL --> Prometheus[Prometheus Metrics]
        end
    end

    subgraph "Infrastructure (Terraform)"
        VPC[VPC/Subnets]
        IAM[IAM Roles]
        EKS[EKS Control Plane]
    end```

---

## Request Flow

User  
→ DNS (Route53)  
→ AWS Application Load Balancer (ALB)  
→ Kubernetes Ingress Controller  
→ Kubernetes Service  
→ Pod  
→ Container  
→ Response (same path back)

- DNS resolves the custom domain to the ALB
- ALB forwards traffic to the Ingress Controller
- Ingress routes traffic based on path rules
- Services forward traffic to healthy Pods
- Pods serve requests through containers

**Failure scenarios:**
- If the Ingress controller is unavailable, external traffic cannot reach the cluster
- If Service selectors are misconfigured, traffic does not reach Pods even if they are running

---

## Why This Stack?

- **Docker**: Packages applications and dependencies consistently
- **Kubernetes (EKS)**: Provides scaling, self-healing, and service discovery
- **EKS**: Eliminates operational overhead of managing the Kubernetes control plane
- **Terraform**: Enables repeatable, version-controlled infrastructure provisioning
- **ALB + Ingress**: Provides Layer-7 routing using AWS-native load balancing
- **GitHub Actions**: Used for CI (build and push container images)
- **Argo CD**: Used for CD via GitOps to prevent configuration drift

AWS IAM Roles for Service Accounts (IRSA) are used to avoid hardcoded
credentials and follow least-privilege access principles.

---

## CI/CD Flow (Example: Product Catalog Service)

1. Code changes trigger a GitHub Actions workflow
2. Docker image is built and pushed to the container registry
3. Kubernetes manifests are updated with the new image tag
4. Argo CD detects the change and synchronizes it to the cluster

This pattern can be reused for other microservices in the platform.

---

## Failure & Debugging Approach

- Pod crash → Kubernetes automatically restarts the Pod
- Node failure → Pods are rescheduled onto healthy nodes
- Kafka failure → Dependent services degrade while others remain available
- Misconfigured environment variables → Pods may CrashLoop or behave incorrectly

Debugging typically starts with:
```bash
kubectl get pods
kubectl describe pod
kubectl logs```

## Cost Control Strategy

Running EKS continuously is expensive, so infrastructure is deployed only
when required and destroyed afterward using Terraform.

For interviews and demonstrations, architecture diagrams, screenshots, and
clear explanations are used instead of keeping live infrastructure running.

## How to Run (High-Level)

Provision infrastructure:
cd terraform && terraform apply

Deploy application:
kubectl apply -f kubernetes/manifests/

## Destroy
cd terraform && terraform destroy

## Documentation

Architecture diagrams and workflow screenshots are available in the `docs/` directory.


