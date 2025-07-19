
## `preferredDuringSchedulingIgnoredDuringExecution` Examples

This affinity type means the scheduler will **try** to find a node that satisfies the specified conditions. If it cannot find such a node, it will still schedule the pod, but on a node that doesn't meet the preferences. Each preference has a `weight` (1-100) to indicate its importance.

### 1. Preferring a Specific Region

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod-region-preferred
spec:
  containers:
  - name: my-container
    image: nginx
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 80
        preference:
          matchExpressions:
          - key: topology.kubernetes.io/region
            operator: In
            values:
            - us-east-1
```
Explanation: The scheduler will try to schedule this pod in the us-east-1 region. If no nodes are available there, it will schedule it elsewhere.

---

**Part 13: Preferred Affinity - Zone Example**

```markdown
### 2. Preferring a Specific Zone

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod-zone-preferred
spec:
  containers:
  - name: my-container
    image: nginx
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 60
        preference:
          matchExpressions:
          - key: topology.kubernetes.io/zone
            operator: In
            values:
            - us-east-1b
```
Explanation: The scheduler will try to schedule this pod in the us-east-1b zone.


---

**Part 14: Preferred Affinity - Custom Label Example**

```markdown
### 3. Preferring Nodes with a Custom Label

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod-custom-label-preferred
spec:
  containers:
  - name: my-container
    image: nginx
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 70
        preference:
          matchExpressions:
          - key: node-role.kubernetes.io/gpu
            operator: Exists
```
Explanation: The scheduler will prefer to schedule this pod on nodes that have the node-role.kubernetes.io/gpu label (indicating a GPU node).


---

**Part 15: Preferred Affinity - High Capacity Example**

```markdown
### 4. Preferring Nodes with More Resources (Example using a custom label indicating capacity)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod-high-capacity-preferred
spec:
  containers:
  - name: my-container
    image: resource-intensive-app
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 90
        preference:
          matchExpressions:
          - key: custom.kubernetes.io/large-instance
            operator: In
            values:
            - "true"
```
Explanation: The scheduler will prefer nodes explicitly labeled as custom.kubernetes.io/large-instance: "true".

---

**Part 16: Preferred Affinity - Multiple Preferences Example**

```markdown
### 5. Combining Multiple Preferences with Different Weights

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod-multi-preferred
spec:
  containers:
  - name: my-container
    image: nginx
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 80
        preference:
          matchExpressions:
          - key: topology.kubernetes.io/region
            operator: In
            values:
            - eu-west-1
      - weight: 50
        preference:
          matchExpressions:
          - key: app-group
            operator: In
            values:
            - data-processing
```
Explanation: The scheduler will strongly prefer nodes in eu-west-1 (weight 80). Additionally, it will have a weaker preference (weight 50) for nodes labeled app-group: data-processing.

---

**Part 17: Preferred Affinity - NotIn Operator Example**

```markdown
### 6. Preferring Nodes that *Don't* Have a Specific Label

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod-prefer-no-dev-nodes
spec:
  containers:
  - name: my-container
    image: production-app
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 75
        preference:
          matchExpressions:
          - key: environment
            operator: NotIn
            values:
            - development
```
Explanation: The scheduler will prefer nodes that are not labeled as environment: development. This is useful for keeping production workloads off development nodes.


---

**Part 18: Preferred Affinity - SSD Example**

```markdown
### 7. Preferring Nodes with Specific Hardware (e.g., SSDs)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod-ssd-preferred
spec:
  containers:
  - name: my-container
    image: database-app
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 95
        preference:
          matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
```
Explanation: This pod will strongly prefer to be scheduled on nodes labeled with disktype: ssd.

---
And Example | OR example

``yaml
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 80 # This is the first "preference block"
        preference:
          # MatchExpressions here define AND/OR logic within THIS preference
          matchExpressions:
          - key: topology.kubernetes.io/region
            operator: In
            values:
            - us-east-1
          - key: disk-type
            operator: Exists # AND condition with the region
      - weight: 50 # This is a separate "preference block"
        preference:
          matchExpressions:
          - key: app-group
            operator: In
            values:
            - data-processing
``


**Part 19: Important Considerations**

```markdown
---

### Important Considerations:
```
* **Labeling Nodes:** For all these examples to work, your Kubernetes nodes must be appropriately labeled. Cloud providers often auto-label with region/zone, but custom labels need to be applied manually or via automation.
* **Operator Types:**
    * `In`: The node's label value must be one of the specified values.
    * `NotIn`: The node's label value must *not* be any of the specified values.
    * `Exists`: The label must exist on the node (value doesn't matter).
    * `DoesNotExist`: The label must *not* exist on the node.
    * `Gt`: Greater than (for numerical string values).
    * `Lt`: Less than (for numerical string values).
* **`IgnoredDuringExecution`**: Both `requiredDuringSchedulingIgnoredDuringExecution` and `preferredDuringSchedulingIgnoredDuringExecution` mean that once a pod is scheduled, if the node's labels change and no longer satisfy the affinity rules, the pod will **not** be evicted or moved. The rules are only evaluated *during scheduling*.
* **Interplay with `nodeSelector`**: While `nodeSelector` offers simpler key-value matching for hard requirements, `nodeAffinity` provides more flexibility with operators and soft preferences. `nodeSelector` and `nodeAffinity` can be used together; if both are specified, *both* must be satisfied for the pod to be scheduled.
* **Taints and Tolerations**: Remember that node affinity (both required and preferred) works *in conjunction* with taints and tolerations. Taints repel pods unless a pod has a corresponding toleration. Affinity *attracts* pods to specific nodes.
```
