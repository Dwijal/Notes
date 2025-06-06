
# Kubernetes LimitRange Cheatsheet

`LimitRange` is a Kubernetes resource used to enforce default resource requests/limits and set constraints within a namespace.

---

## 🔧 Purpose
Ensure that pods/containers do not consume excessive resources and have sane defaults.

---

## 📄 LimitRange YAML Example

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: resource-limits
  namespace: dev
spec:
  limits:
  - default:
      cpu: "500m"
      memory: "512Mi"
    defaultRequest:
      cpu: "200m"
      memory: "256Mi"
    max:
      cpu: "1"
      memory: "1Gi"
    min:
      cpu: "100m"
      memory: "128Mi"
    type: Container
```

---

## 🧱 Fields Explained

| Field           | Description |
|----------------|-------------|
| `default`       | Default resource limits applied to containers **if none specified** |
| `defaultRequest`| Default resource requests used for scheduling **if none specified** |
| `max`           | Maximum resource usage allowed |
| `min`           | Minimum resource request/limit required |
| `type`          | Target resource type: `Container`, `Pod`, or `PersistentVolumeClaim` |

---

## 📦 Supported Types

- `Container` (most common)
- `Pod`
- `PersistentVolumeClaim`

---

## 🚀 Key Notes

- Applies **only within a namespace**
- Does **not override explicitly defined** values in pods
- Helps avoid resource hogs and enforces baseline fairness

---

## 📌 Commands

```bash
# Create LimitRange from file
kubectl apply -f limitrange.yaml

# View LimitRanges in a namespace
kubectl get limitranges -n <namespace>

# Describe a specific LimitRange
kubectl describe limitrange <name> -n <namespace>
```

---

## 📚 Best Practices

- Always define `defaultRequest` to ensure proper scheduling
- Use `max`/`min` to cap and protect cluster resources
- Combine with `ResourceQuota` for stricter enforcement
