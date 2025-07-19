### Kubernetes Node Affinity Examples: `requiredDuringScheduling` vs. `preferredDuringScheduling`


---

## `requiredDuringSchedulingIgnoredDuringExecution` Examples

This affinity type means the pod **must** be scheduled on a node that satisfies **all** the specified conditions. If no such node exists, the pod will remain unscheduled.

### 1. Matching by Region (Standard Label)

Using the `topology.kubernetes.io/region` label, often automatically set by cloud providers.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod-region-required
spec:
  containers:
  - name: my-container
    image: nginx
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: topology.kubernetes.io/region
            operator: In
            values:
            - us-east-1
```
Explanation: This pod will only be scheduled on nodes that have the label topology.kubernetes.io/region with a value of us-east-1.


---

**Part 4: Required Affinity - Zone Example**

```markdown
### 2. Matching by Zone (Standard Label)

Using the `topology.kubernetes.io/zone` label, also commonly set by cloud providers.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod-zone-required
spec:
  containers:
  - name: my-container
    image: nginx
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: topology.kubernetes.io/zone
            operator: In
            values:
            - us-east-1a
```
Explanation: This pod will only be scheduled on nodes within the us-east-1a zone.


---

**Part 5: Required Affinity - Custom Label Example**

```markdown
### 3. Matching by Custom Label (General/Normal Label)

Using a user-defined label.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod-custom-label-required
spec:
  containers:
  - name: my-container
    image: nginx
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: app-type
            operator: In
            values:
            - backend
```
Explanation: This pod will only be scheduled on nodes that have the label app-type with a value of backend.

---

**Part 6: Required Affinity - Node Name Example**

```markdown
### 4. Matching by Node Name

While `nodeAffinity` is primarily for labels, `nodeName` is a direct way to target a specific node and acts as a hard requirement.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod-specific-node
spec:
  nodeName: my-specific-node-name
  containers:
  - name: my-container
    image: nginx
```
Explanation: This pod will only be scheduled on the node named my-specific-node-name. If that node doesn't exist or isn't ready, the pod won't schedule.

---

**Part 7: Required Affinity - Multiple `matchExpressions` (AND) Example**

```markdown
### 5. Multiple `matchExpressions` (AND logic)

When multiple `matchExpressions` are specified within the same `nodeSelectorTerm`, they are logically ANDed.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod-multi-required
spec:
  containers:
  - name: my-container
    image: nginx
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: topology.kubernetes.io/region
            operator: In
            values:
            - us-west-2
          - key: disk-type
            operator: Exists
```
Explanation: This pod requires nodes that are in us-west-2 AND have the disk-type label defined (regardless of its value).



---

**Part 8: Required Affinity - Multiple `nodeSelectorTerms` (OR) Example**

### 6. Multiple `nodeSelectorTerms` (OR logic)

When multiple `nodeSelectorTerms` are specified, they are logically ORed. The pod will be scheduled if it matches *any* of the `nodeSelectorTerms`.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod-or-required
spec:
  containers:
  - name: my-container
    image: nginx
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: app-role
            operator: In
            values:
            - webserver
        - matchExpressions:
          - key: environment
            operator: In
            values:
            - production
```
Explanation: This pod requires nodes that are either webserver role OR in the production environment.

---

**Part 9: Required Affinity - `DoesNotExist` Operator Example**

```markdown
### 7. Using `DoesNotExist` Operator

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod-no-gpu
spec:
  containers:
  - name: my-container
    image: my-gpu-unfriendly-app
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: accelerator
            operator: DoesNotExist
```
Explanation: This pod will only be scheduled on nodes that do not have the accelerator label. Useful for preventing certain workloads from landing on specialized hardware.


---

**Part 10: Required Affinity - `Gt`/`Lt` Operators Example**

```markdown
### 8. Using `Gt` (Greater Than) and `Lt` (Less Than) Operators (for numerical labels)

These operators are generally used with numerical string values.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod-high-cpu
spec:
  containers:
  - name: my-container
    image: cpu-intensive-app
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: custom.kubernetes.io/cpu-count
            operator: Gt
            values:
            - "8"
```
Explanation: This pod will only be scheduled on nodes that have a custom.kubernetes.io/cpu-count label with a value greater than 8 (as a string comparison, so ensure consistency).

