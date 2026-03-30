# Kubernetes Cluster Security & Hardening

Securing a Kubernetes (K8s) cluster is significantly more complex than securing a standalone Docker host. A single compromised Pod can often lead to full cluster takeover (node-level escalation, secret extraction).

## 1. Network Policies (Zero Trust Egress)

By default, any pod can communicate with any other pod across namespaces in Kubernetes. Implement default-deny Network Policies.

> [!WARNING]
> This will break applications that don't have explicit allow-rules defined. Test egress in staging.

```yaml
# default-deny-all.yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: default-deny-all
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

## 2. Pod Security Admission (Replaces PSPs)

Enforce strict non-root policies across your cluster namespaces by applying the `restricted` profile to production namespaces. This prevents pods from invoking `hostPath`, dropping capabilities, or running as root.

```yaml
# Applying directly to a namespace
apiVersion: v1
kind: Namespace
metadata:
  name: hardened-prod
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/enforce-version: latest
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/audit-version: latest
    pod-security.kubernetes.io/warn: restricted
    pod-security.kubernetes.io/warn-version: latest
```

## 3. RBAC Auditing

Ensure your CI/CD service accounts or human operators do not hold the `cluster-admin` RoleBinding. The most dangerous permissions operators mistakenly grant are:

1. `create pods` or `create deployments` (leads to spinning up privileged pods).
2. `get secrets` (stealing API tokens or TLS certs).
3. `impersonate` (assuming control of a higher-level ServiceAccount).

Use `kubectl-who-can` (a highly recommended plugin) to map these visually.
```bash
kubectl auth can-i create pods --as=system:serviceaccount:default:my-app
# Output: yes / no
```

## 4. OPA Gatekeeper for Governance

Define security policies mathematically via Rego. Write a constraint template that explicitly bans `:latest` image tags in production deployments to prevent supply-chain poisoning.

> [!TIP]
> Never assume the cluster will reject a bad deployment. Always write Gatekeeper rules to actively reject Pods that mount the `hostNetwork`.

```yaml
# Gatekeeper Constraint Example
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: must-have-owner
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Namespace"]
  parameters:
    labels: ["owner"]
```
