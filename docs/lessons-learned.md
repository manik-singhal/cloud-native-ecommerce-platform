# Lessons Learned (What Actually Changed How I Think)

This project was not about learning new tools.
It was about unlearning assumptions and understanding how small decisions
compound in real systems.

Below are the most important lessons that genuinely changed how I approach
DevOps and platform engineering.

---

## 1. Docker: Smaller Images Are an Engineering Choice, Not an Optimization Later

### What I learned
Using multi-stage Docker builds drastically reduces image size by separating
build-time dependencies from runtime artifacts.

Earlier, I treated Dockerfiles as something that just “needs to work”.
Now I understand that image size directly affects:
- Pull time
- Startup latency
- CI speed
- Resource usage in the cluster

### Why this mattered
When running multiple microservices, even small inefficiencies multiply.
Multi-stage builds forced me to think about **what actually needs to ship**
versus what is only needed to build.

This shifted my mindset from “make it run” to “make it efficient by design”.

---

## 2. Terraform: Modularity Is Not Optional at Scale

### What I learned
Terraform modules are not just for cleanliness — they are for control.

Initially, writing everything in a single Terraform file felt faster.
But as infrastructure grew (VPC, EKS, IAM, node groups),
the lack of structure quickly became a liability.

### Why this mattered
Modular Terraform:
- Makes infrastructure reusable
- Reduces blast radius of changes
- Allows teams to reason about one component at a time

I now see Infrastructure as Code as **software**, not configuration.
If it’s not modular, it doesn’t scale — mentally or operationally.

---

## 3. Kubernetes: Environment Variables Are the Real Interface

### What I learned
Kubernetes manifests are not just YAML files — they are the contract between
infrastructure and application.

Environment variables turned out to be one of the most critical parts of that
contract, and also one of the easiest places to break things.

### Why this was hard
Misconfigured environment variables caused:
- Pods that were Running but not working
- Silent failures
- Debugging sessions that initially made no sense

Kubernetes didn’t restart Pods because, technically, nothing was “crashing”.

### What changed for me
I now understand that:
- Kubernetes only manages containers
- Application correctness still depends on configuration

This forced me to think beyond Pod status and focus on **application behavior**.

---

## 4. CI/CD: Automation Fails Silently If the Platform Isn’t Enabled

### What I learned
CI pipelines don’t just run because YAML exists.

For GitHub Actions to execute workflows, the repository must explicitly allow
GitHub Actions to run.

### Why this mattered
This was a simple but humbling lesson:
Automation depends on **platform-level permissions**, not just code.

It taught me to always check:
- Repository settings
- Execution permissions
- Platform defaults

Before assuming the pipeline logic is broken.

---

## 5. Kubernetes: Resource Requests & Limits Are About Fairness, Not Perfection

### What I learned
Resource requests and limits exist to protect the cluster, not to perfectly optimize an application. Requests inform the scheduler about expected resource usage, while limits prevent a single Pod from consuming more than its fair share.

### Why this mattered
In shared clusters, missing or incorrect limits can allow one service to impact others on the same node. I observed that when limits are set too low, Pods may get **OOMKilled** even though the application logic itself is correct. Kubernetes is enforcing platform constraints, not evaluating application health.

### How this changed my thinking
In this project, resource requests and limits are defined across all manifests to reflect production intent. However, I did not aggressively tune them because this is a demo workload, not a traffic-tested system. The real value was understanding that resource configuration is about **reducing blast radius and maintaining cluster stability**, not chasing perfect numbers.

---

## Final Reflection

The biggest takeaway from this project is that DevOps is not about knowing
more tools — it’s about understanding **where systems fail quietly**.

Most issues I faced were not complex bugs.
They were small misconfigurations with large effects.

This project shifted my thinking from:
“Why is this not working?”
to
“What assumption did I just make that isn’t true?”
