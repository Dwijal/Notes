
# Kubernetes Probes Cheat Sheet with Deployment Example

This guide covers all three types of Kubernetes health probes — `startupProbe`, `livenessProbe`, and `readinessProbe` — along with their parameters and a complete example in a Deployment YAML.

---

## 🔍 Probe Types

| Probe Type      | Purpose                                                                 |
|------------------|-------------------------------------------------------------------------|
| `startupProbe`   | Used to check if the application has started. Disables other probes until it succeeds. |
| `livenessProbe`  | Checks if the application is alive. If it fails, the container is restarted. |
| `readinessProbe` | Checks if the application is ready to serve traffic. If it fails, the Pod is removed from the Service endpoints. |

---

## 🔧 Probe Actions

| Action       | Description                                   |
|--------------|-----------------------------------------------|
| `httpGet`    | Probe uses an HTTP GET request                |
| `tcpSocket`  | Probe opens a TCP connection on the port      |
| `exec`       | Probe runs a command inside the container     |

---

## 🛠️ Common Parameters

| Parameter            | Description                                                                 |
|----------------------|-----------------------------------------------------------------------------|
| `initialDelaySeconds`| Seconds to wait before starting the first probe                             |
| `periodSeconds`      | How often to perform the probe (in seconds)                                 |
| `timeoutSeconds`     | Time to wait for probe response before failure                              |
| `successThreshold`   | Number of successes required to consider the probe successful               |
| `failureThreshold`   | Number of failures before marking the probe as failed                       |

---

## 📦 Deployment Example Using All Probes

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: probe-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: probe-demo
  template:
    metadata:
      labels:
        app: probe-demo
    spec:
      containers:
      - name: demo
        image: hashicorp/http-echo
        args:
          - "-text=ok"
          - "-listen=:8080"
        ports:
        - containerPort: 8080

        startupProbe:
          httpGet:
            path: /start
            port: 8080
          initialDelaySeconds: 0
          periodSeconds: 5
          timeoutSeconds: 1
          successThreshold: 1
          failureThreshold: 3

        livenessProbe:
          httpGet:
            path: /live
            port: 8080
          initialDelaySeconds: 3
          periodSeconds: 5
          timeoutSeconds: 1
          successThreshold: 1
          failureThreshold: 3

        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 3
          periodSeconds: 5
          timeoutSeconds: 1
          successThreshold: 1
          failureThreshold: 3
```

---

✅ **Tip**: `startupProbe` disables `liveness` and `readiness` checks until it succeeds. Once successful, `liveness` and `readiness` begin checking.


```
Behavior of RP, LP, and Startup Probes with Multi-Container Pods
A pod in Kubernetes can host multiple containers.
The pod's status reflects the worst state of any container within it

- Startup Probe:

Each container's startup probe runs independently.
If the startup probe for one container fails, only that container is affected, and liveness/readiness probes are not triggered for it. Other containers proceed unaffected.

- Readiness Probe:

Readiness probes determine whether the pod is ready to serve traffic.
If the readiness probe fails for one container, the entire pod is marked "Not Ready" to ensure no partial or unreliable service is provided.

- Liveness Probe:

Liveness probes are container-specific.
If the liveness probe fails for one container, only that container is restarted by the Kubelet, while other containers remain unaffected.
```


