

### If we just write the policyType then both ingress and egress traffic is blocked. 
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: full-secure-policy
  namespace: secure-apps
spec:
  podSelector:
    matchLabels:
      role: frontend   # Target: Pods with label role=frontend
  policyTypes:
    - Ingress
    - Egress
```

### Below policy convey we have defined the ingress rule but not defined the egress so it allows the egress traffic as well.

```
# Backend policy - allow ingress only from frontend pods
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-ingress
spec:
  podSelector:
    matchLabels:
      role: backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
  policyTypes:
  - Ingress
```

### No podSelector = select all pods in namespace
```
podSelector: {}
```

### Now no pod can accept ingress and egress unless allowed explicitly.
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress-egress
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress

```

### Commonly, allow pods to connect to cluster DNS:
```
egress:
- to:
  - namespaceSelector:
      matchLabels:
        kubernetes.io/kube-system: "true"
    podSelector:
      matchLabels:
        k8s-app: kube-dns
  ports:
  - protocol: UDP
    port: 53
```

### Complete NetworkPolicy Example

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: full-secure-policy
  namespace: secure-apps
spec:
  podSelector:
    matchLabels:
      role: frontend   # Target: Pods with label role=frontend
  policyTypes:
    - Ingress
    - Egress

  # 👇 Ingress Rules (Incoming traffic TO the pod)
  ingress:
    # Allow from:
    # (1) Pods in same namespace with role=backend
    # OR
    # (2) Pods in namespaces labeled env=prod with app=db
    # OR
    # (3) IP block for known external source
    - from:
        - podSelector:
            matchLabels:
              role: backend
        - namespaceSelector:
            matchLabels:
              env: prod
          podSelector:
            matchLabels:
              app: db
        - ipBlock:
            cidr: 10.0.0.0/8
            except:
              - 10.10.10.0/24   # block suspicious subnet
      ports:
        - protocol: TCP
          port: 443    # Allow HTTPS only
        - protocol: TCP
          port: 8443   # Allow internal secure admin

  # 👇 Egress Rules (Outgoing traffic FROM the pod)
  egress:
    # Allow DNS resolution
    - to:
        - ipBlock:
            cidr: 0.0.0.0/0
      ports:
        - protocol: UDP
          port: 53
    # Allow API requests to app labeled api-service in any namespace
    - to:
        - podSelector:
            matchLabels:
              app: api-service
      ports:
        - protocol: TCP
          port: 443
    # Allow NTP for time sync
    - to:
        - ipBlock:
            cidr: 192.168.1.100/32
      ports:
        - protocol: UDP
          port: 123

```
