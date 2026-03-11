---
title: "Practical Kubernetes debugging tips I use daily"
date: 2026-03-11
draft: false
---

After spending a significant amount of time operating services on Kubernetes, I have developed a small set of habits that make debugging faster and less frustrating.

#### Start with pod state, not logs

When something is wrong, the first instinct is to jump to logs. But logs only exist if the container is actually running. Start with:

```bash
kubectl get pods -n <namespace>
kubectl describe pod <pod-name> -n <namespace>
```

`describe` gives you events, which often contain the actual root cause before a line of application log has been written.

#### Events are underrated

```bash
kubectl get events -n <namespace> --sort-by='.lastTimestamp'
```

Events show OOMKills, failed pulls, scheduling failures, and readiness probe issues — all things that do not necessarily appear in application logs.

#### Use `--previous` for crash loops

If a container is in `CrashLoopBackOff`, the current container's logs may be empty. Use:

```bash
kubectl logs <pod-name> -n <namespace> --previous
```

This fetches logs from the last terminated container instance, which usually contains the actual error.

#### Ephemeral debug containers

For a running pod with a minimal image (no shell, no tools), you can attach a debug container without restarting:

```bash
kubectl debug -it <pod-name> -n <namespace> --image=busybox --target=<container-name>
```

This is invaluable when you need to inspect the network or filesystem of a running container without modifying the deployment.

These four habits alone cover the majority of production debugging scenarios I encounter. The rest is just reading carefully.
