# Docker Host & Container Security Hardening

Running a vulnerable Docker container is the fastest way a server gets compromised. Developers consistently run containers with `root` privileges. This cheatsheet contains professional infrastructure mitigation techniques.

## 1. Eliminate Root Escalation

Never run your container as the root user. If your application doesn't strictly need root access (e.g., binds to ports < 1024), create and run an unprivileged user.

```dockerfile
# Dockerfile snippet
RUN addgroup -g 1000 appgroup && \
    adduser -u 1000 -G appgroup -D appuser
USER appuser
CMD ["node", "server.js"]
```

## 2. Kernel Capabilities

A default Docker container holds unnecessary access to the host kernel (CAP_CHOWN, CAP_KILL). Strip them all, and strictly whitelist what is required.

> [!TIP]
> Use `--cap-drop=all` as part of your CI/CD deployment logic by default unless the container fundamentally requires root networking execution.

### Secure Execution
```bash
docker run -d --cap-drop=all --cap-add=NET_BIND_SERVICE nginx:latest
```

## 3. Read-Only Core Filesystem

Mitigate an active exploit (like RCE or web shell uploading) by forcing the container filesystem to become Read-Only.

> [!IMPORTANT]
> A compromised container without write permissions blocks 99% of automated scripts from executing logic that modifies the operating system payload.

```bash
docker run --read-only --tmpfs /tmp:rw,noexec,nosuid my-image
```
*Note: Any directory the application explicitly needs to write to must be mounted as a `tmpfs` or standard volume.*

## 4. Resource Limitations

Denial-of-Service attacks or simple memory leaks will take down the entire host unless you constrain the container.

- Memory: `--memory="512m"`
- CPU: `--cpus=".5"`
- Default Process Limit (pids): `--pids-limit 100`

```bash
docker run -d --memory="512m" --cpus="0.5" --pids-limit 100 redis:alpine
```
