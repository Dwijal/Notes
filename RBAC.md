
# Kubernetes RBAC Reference

This file contains basic definitions and commands for managing Roles, ClusterRoles, RoleBindings, and ClusterRoleBindings in Kubernetes.

---

## Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

## ClusterRole

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

## RoleBinding (Azure AD Example)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: "aad_user_object_id"  # Replace with the Azure AD user object ID
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

## ClusterRoleBinding (Azure AD Example)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-cluster-pods
subjects:
- kind: Group
  name: "aad_group_object_id"  # Replace with the Azure AD group object ID
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-pod-reader
  apiGroup: rbac.authorization.k8s.io
```

---

## Kubernetes Verbs

These are common verbs used in Kubernetes RBAC:

- **get** – Read a resource
- **list** – List resources
- **watch** – Watch for changes to resources
- **create** – Create a new resource
- **update** – Modify an existing resource
- **patch** – Partially update a resource
- **delete** – Remove a resource
- **deletecollection** – Delete a collection of resources
- **impersonate** – Act as another user, group, or service account
- **bind** – Bind a role to a user/group/service account
- **escalate** – Allow privilege escalation

> Full list of verbs may vary based on the API server and resources involved.

---

## Useful kubectl Commands

```bash
# Create from file
kubectl apply -f role.yaml
kubectl apply -f clusterrole.yaml
kubectl apply -f rolebinding.yaml
kubectl apply -f clusterrolebinding.yaml

# View resources
kubectl get roles -n default
kubectl get clusterroles
kubectl get rolebindings -n default
kubectl get clusterrolebindings

# Describe resources
kubectl describe role pod-reader -n default
kubectl describe clusterrole cluster-pod-reader

# Delete resources
kubectl delete role pod-reader -n default
kubectl delete clusterrole cluster-pod-reader
kubectl delete rolebinding read-pods -n default
kubectl delete clusterrolebinding read-cluster-pods
```
