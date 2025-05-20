
# Using Azure Key Vault Secrets in AKS via Secrets Store CSI Driver and Environment Variables

This guide explains how to use Azure Key Vault secrets inside Azure Kubernetes Service (AKS) pods using Secrets Store CSI Driver with syncing to Kubernetes Secrets for environment variable injection.

---

## Step 1: Enable Secrets Store CSI Driver Addon in AKS

If AKS cluster is not created:

```
az aks create -n <aks-cluster-name> -g <resource-group-name> --enable-addons azure-keyvault-secrets-provider --enable-managed-identity
```

If AKS cluster is already created:

```
az aks enable-addons --addons azure-keyvault-secrets-provider --name <aks-cluster-name> --resource-group <resource-group-name>
```

---

## Step 2: Create a User Assigned Managed Identity (UAMI)

```
az identity create -g <resource-group-name> -n <uami-name>
```

Note down the `clientId` and `resourceId`.

---

## Step 3: Assign Access to Key Vault

Create a Key Vault:

```
az keyvault create -n <keyvault-name> -g <resource-group-name>
```

Add a secret to Key Vault:

```
az keyvault secret set --vault-name <keyvault-name> -n mysql-password --value "<secure-pass>"
```

Assign **Key Vault Secrets User** role to the UAMI:

```
az role assignment create --assignee <clientId-of-uami> --role "Key Vault Secrets User" --scope /subscriptions/<sub-id>/resourceGroups/<rg>/providers/Microsoft.KeyVault/vaults/<vault-name>
```

---

## Step 4: Assign UAMI to Node Pool (Kubelet Identity)

```
az aks nodepool update   --resource-group <resource-group-name>   --cluster-name <aks-cluster-name>   --name <nodepool-name>   --assign-identity <uami-resource-id>
```

> NOTE: The identity is used by the kubelet to access the Key Vault.

---

## Step 5: Create SecretProviderClass

```yaml
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: azure-kvname-sync-env
spec:
  provider: azure
  secretObjects:
  - secretName: kv-synced-secret
    type: Opaque
    data:
    - objectName: mysql-password
      key: mysql-password
  parameters:
    usePodIdentity: "false"
    useVMManagedIdentity: "true"
    userAssignedIdentityID: "<client-id-of-uami>"
    keyvaultName: <keyvault-name>
    cloudName: "AzurePublicCloud"
    objects: |
      array:
        - |
          objectName: mysql-password
          objectType: secret
          objectVersion: ""
    tenantId: <tenant-id>
```

---

## Step 6: Pod Spec Using Synced Secret as Environment Variable

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-test
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "echo MYSQL_PASSWORD=$MYSQL_PASSWORD && sleep 3600"]
    env:
    - name: MYSQL_PASSWORD
      valueFrom:
        secretKeyRef:
          name: kv-synced-secret
          key: mysql-password
  volumes:
  - name: secrets-store-inline
    csi:
      driver: secrets-store.csi.k8s.io
      readOnly: true
      volumeAttributes:
        secretProviderClass: "azure-kvname-sync-env"
```

---

## Step 7: Validate Access

List secrets mounted via volume (optional):

```
kubectl exec secret-env-test -- ls /mnt/secrets-store/
```

Check the synced secret:

```
kubectl get secret kv-synced-secret -o yaml
```

---
## TO sync the secrets from key vault to the kubernetes secrets and use it as env variables.
```
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: azure-kvname-sync
spec:
  provider: azure
  parameters:
    usePodIdentity: "false"
    useVMManagedIdentity: "true" # Set to true for using managed identity both System and User Assigned
    userAssignedIdentityID: "<client-id-of-uami>"  #  If empty, then defaults to use the system assigned identity on the VM/Else Provide the Client ID of the UMI
    keyvaultName: aksdemocluster-kv
    cloudName: "AzurePublicCloud"
    objects: |
      array:
        - |
          objectName: mysql-password
          objectType: secret
          objectVersion: ""
    tenantId: <your-tenant-id>
  secretObjects:
  - secretName: kv-synced-secret
    type: Opaque
    data:
      - objectName: mysql-password
        key: mysql-password
```

## Summary

- Azure Key Vault stores the secret.
- Secrets Store CSI Driver mounts it via volume and syncs it to a Kubernetes Secret.
- Pod consumes the Kubernetes Secret via environment variable.

This approach is secure, avoids hardcoding secrets, and allows Key Vault-backed secret rotation.
