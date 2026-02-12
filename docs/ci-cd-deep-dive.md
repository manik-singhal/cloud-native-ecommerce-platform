# CI/CD Deep Dive

This document explains how CI and CD are designed and used in this project,
and why they are intentionally separated.

The goal of this setup is reliability, controlled change, and easy recovery
when something goes wrong.

---

## Why CI ≠ CD

CI and CD solve different problems.

**CI (Continuous Integration)** answers:
> “Is this change buildable and safe to promote?”

**CD (Continuous Deployment)** answers:
> “Should this change be running in the cluster right now?”

Combining both into a single pipeline increases risk because:
- CI needs access to source code and registries
- CD needs access to the cluster
- Failures become harder to isolate

In this project, CI and CD are separated to reduce blast radius and make
failures easier to reason about.

---

## Why GitHub Actions Is Used for CI

GitHub Actions is well-suited for CI because it:
- Integrates directly with pull requests and commits
- Provides fast feedback on every change
- Is ephemeral and stateless
- Does not need access to the Kubernetes cluster

In this project, GitHub Actions is responsible for:
- Building Docker images
- Running basic checks
- Pushing images to the container registry

Once CI completes, its job is done.
It does not deploy anything.

This keeps CI simple and safe.

---

## Why Argo CD Is Used for CD

Argo CD is used for CD because it:
- Runs inside the Kubernetes cluster
- Continuously compares Git state with cluster state
- Automatically reconciles drift
- Supports controlled rollbacks

Unlike push-based CD, Argo CD is **pull-based**.
The cluster pulls desired state from Git instead of Git pushing changes into the cluster.

This means:
- Cluster credentials never leave the cluster
- Deployment logic stays close to the runtime environment
- Manual changes in the cluster are detected and corrected

---

## What Happens When Things Break

### If CI Fails

- No new image is built or promoted
- Kubernetes and running services are unaffected
- The system remains stable

CI failures block **change**, not **availability**.

---

### If CD Is Paused or Argo CD Is Down

- Running applications continue to work normally
- No new changes are deployed
- The cluster stops reconciling drift

This is safe by design.
CD failure does not bring the system down — it only freezes deployment.

---

### If Git Is Wrong

- Argo CD deploys incorrect configuration
- Issues surface quickly and consistently across environments

Because Git is the single source of truth:
- The root cause is always visible
- Rollback is a `git revert`
- There is no hidden state to hunt for

Wrong Git is recoverable.
Unknown cluster state is not.

---

## Why GitOps Reduces Blast Radius

GitOps reduces blast radius by:
- Forcing all changes through version control
- Making deployments deterministic
- Preventing manual, untracked changes
- Enabling fast rollback using Git history

Instead of debugging “what changed in the cluster,”
the question becomes:
> “Which commit introduced this behavior?”

This keeps failures:
- Smaller
- Easier to diagnose
- Easier to reverse

---

## Summary

- CI validates and packages changes
- CD deploys and maintains cluster state
- Git is the single source of truth
- Failures are isolated by design

This separation makes the system easier to operate,
debug, and trust in production-like environments.
