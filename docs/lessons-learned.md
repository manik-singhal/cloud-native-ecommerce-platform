# Lessons Learned

This project taught me more about where assumptions break than about which tools to use.

The lessons below genuinely changed how I think about infrastructure and deployment workflows.

---

## Docker: Image Size Matters From Day One

### What I learned

Multi-stage Docker builds separate build dependencies from runtime artifacts. This keeps final images small by only shipping what actually needs to run.

I used to think of Dockerfiles as "get it working first, optimize later." But image size affects pull times, startup speed, CI duration, and cluster resource usage. When running multiple microservices, these small inefficiencies add up fast.

### Why this mattered

Multi-stage builds forced me to think about what actually needs to ship versus what's only needed to compile. This shifted my mindset from "make it run" to "make it efficient from the start."

---

## Terraform: Modularity Isn't Optional

### What I learned

Terraform modules aren't just for organization—they're for control.

Writing everything in one file felt faster at first. But as infrastructure grew (VPC, EKS, IAM, node groups), that lack of structure became a problem. Changes became riskier because everything was coupled.

### Why this mattered

Modular Terraform makes infrastructure reusable, reduces the blast radius of changes, and lets you reason about one piece at a time.

I now treat Infrastructure as Code like software. If it's not modular, it doesn't scale—mentally or operationally.

---

## Kubernetes: Environment Variables Break Things Silently

### What I learned

Kubernetes manifests define the contract between infrastructure and application. Environment variables are a critical part of that contract—and the easiest part to get wrong.

### Why this was hard

Misconfigured environment variables caused pods that showed "Running" but weren't actually working. The application failed silently because Kubernetes had no reason to restart the pod. Technically, nothing was crashing.

### What changed

I learned that Kubernetes only manages containers. Application correctness still depends on configuration. This forced me to stop relying on pod status and start thinking about application behavior.

---

## CI/CD: Automation Needs Platform Permissions

### What I learned

CI pipelines don't run just because the YAML exists. GitHub Actions workflows require repository-level permissions to execute.

### Why this mattered

This was a simple but humbling lesson. Automation depends on platform settings, not just code. I learned to check repository settings, execution permissions, and platform defaults before assuming the pipeline logic is broken.

---

## Kubernetes: Resource Limits Are About Cluster Health, Not App Optimization

### What I learned

Resource requests and limits protect the cluster, not the application. Requests tell the scheduler what a pod needs. Limits prevent one pod from consuming all node resources.

### Why this mattered

When limits are set too low, pods get OOMKilled even if the application logic is fine. Kubernetes is enforcing cluster-level fairness, not judging application quality.

### What I changed

I defined resource requests and limits across all manifests to reflect production intent. But I didn't tune them aggressively because this is a demo workload, not a traffic-tested system. The real value was understanding that resource configuration is about maintaining cluster stability, not chasing perfect numbers.

---

## What I Took Away

The biggest lesson from this project: DevOps isn't about knowing more tools. It's about understanding where systems fail quietly.

Most issues I hit weren't complex bugs. They were small misconfigurations with large effects.

This project changed how I debug. Instead of asking "why isn't this working," I now ask "what assumption did I just make that isn't true?"
