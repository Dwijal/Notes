
# 🧠 Kubernetes ResourceQuota Cheatsheet

## ✅ What Is a ResourceQuota?
A ResourceQuota is a Kubernetes object that sets **limits on resource usage** per namespace.

It ensures teams or workloads don’t consume excessive cluster resources.

---

## 📦 What Can You Limit?

### 1. Compute Resources
- `requests.cpu`, `requests.memory`
- `limits.cpu`, `limits.memory`

### 2. Object Counts
- `pods`, `services`, `secrets`, `configmaps`
- `replicationcontrollers`, `resourcequotas`, `persistentvolumeclaims`

### 3. Storage Resources
- `requests.storage`, `persistentvolumeclaims`
- `requests.ephemeral-storage`, `limits.ephemeral-storage`

---

## 📜 Sample ResourceQuota YAML

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: "4Gi"
    limits.cpu: "8"
    limits.memory: "8Gi"
    persistentvolumeclaims: "5"
    requests.storage: "20Gi"
```

### 🔍 Explanation:
- Max **10 pods**
- CPU Requests ≤ **4 cores**
- Memory Requests ≤ **4GiB**
- CPU Limits ≤ **8 cores**
- Memory Limits ≤ **8GiB**
- Max **5 PVCs**
- Total Storage ≤ **20GiB**

---

## 💡 How It Works:
- Kubernetes adds up all requests/limits in the namespace.
- If new pod would exceed the quota → it’s rejected.

### Example Error:
```
pods "nginx" is forbidden: exceeded quota: dev-quota, requested: cpu=500m, used: cpu=4, limited: cpu=4
```

---

## 🧪 Useful Commands

Describe quota:
```bash
kubectl describe quota -n <namespace>
```

List all quotas:
```bash
kubectl get resourcequota -n <namespace>
```

