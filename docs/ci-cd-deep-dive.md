# CI/CD Deep Dive

This document explains why CI and CD are separated in this project and how that separation makes deployments safer and easier to debug.

---

## Why CI and CD Are Separate

CI and CD solve different problems.

**CI (Continuous Integration)** answers:
> "Is this change safe to build and package?"

**CD (Continuous Deployment)** answers:
> "Should this be running in the cluster right now?"

Combining them into one pipeline increases risk. CI needs access to source code and image registries. CD needs access to the cluster. When both live in the same pipeline, credential leaks become more dangerous and failures become harder to isolate.

Separating them reduces the blast radius. If CI is compromised, it can't touch the running cluster. If CD goes down, CI can still build and validate changes.

---

## Why GitHub Actions for CI

GitHub Actions runs outside the cluster and integrates directly with the repository. This makes it well-suited for CI because:

- It triggers automatically on pull requests and commits
- It provides fast feedback on every code change
- It's ephemeral—no persistent state to manage
- It doesn't need Kubernetes credentials

In this project, GitHub Actions:
- Builds Docker images
- Runs code quality checks
- Pushes images to the container registry

Once CI finishes, it's done. It doesn't deploy anything. This keeps the pipeline simple and the cluster isolated.

---

## Why ArgoCD for CD

ArgoCD runs inside the Kubernetes cluster. It continuously watches Git for changes and reconciles the cluster state to match.

Unlike push-based CD (where pipelines push changes into the cluster), ArgoCD is **pull-based**. The cluster pulls the desired state from Git rather than having Git push into the cluster.

This means:
- Cluster credentials never leave the cluster
- Deployment logic runs close to where services actually run
- Manual changes to the cluster are detected and corrected automatically

ArgoCD also supports rollbacks through Git history, which makes recovery straightforward.

---

## What Happens When Things Break

### If CI Fails

When CI fails:
- No new image gets built
- No changes are promoted
- Running services in Kubernetes are completely unaffected

CI failures block **new changes**, not **availability**. The system stays stable.

---

### If CD Fails or ArgoCD Goes Down

When CD stops working:
- Applications that are already running continue to work normally
- New deployments stop
- The cluster stops reconciling drift

This is safe by design. CD failure doesn't bring the system down—it just pauses new deployments until CD is restored.

---

### If Git Contains Bad Configuration

If someone commits broken configuration to Git:
- ArgoCD deploys it (because Git is the source of truth)
- The issue becomes visible immediately

Because all changes flow through Git:
- The root cause is always traceable to a commit
- Rollback is a simple `git revert`
- There's no hidden cluster state to debug

Bad Git config is recoverable. Unknown cluster state is not.

---

## Why GitOps Reduces Risk

GitOps reduces risk by:
- Requiring all changes to go through version control
- Making deployments deterministic (same input = same result)
- Preventing manual, untracked changes to the cluster
- Enabling fast rollback using Git history

Instead of asking "what changed in the cluster," the question becomes:
> "Which commit caused this behavior?"

This makes failures:
- Easier to trace
- Faster to diagnose
- Simpler to reverse

---

## Validation

The CI/CD separation was validated through actual usage.

### CI Pipeline

GitHub Actions successfully builds images, runs checks, and pushes to the registry without any cluster access.

![CI Pipeline Execution](../screenshots/cicd/github-actions-ci.png)

### CD Sync

ArgoCD continuously monitors Git and keeps the cluster synchronized with the desired state.

![ArgoCD Sync Status](../screenshots/cicd/argocd-app-sync.png)

---

## Summary

- CI builds and validates changes
- CD deploys and maintains cluster state
- Git is the single source of truth
- Failures are isolated by design

This separation makes the system easier to operate and debug.
