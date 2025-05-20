
# AKS Pod Identity vs Workload Identity

This markdown explains what Pod Identity is, how it is implemented in Azure Kubernetes Service (AKS), its limitations, and how Workload Identity addresses those limitations.

## 🔷 What is Azure Pod Identity?

Azure Pod Identity allows Kubernetes pods to access Azure resources securely using Azure Active Directory (AAD) identities — without managing credentials in code or config files.

### How It Works:

- Pods are assigned Azure managed identities (usually UAMI).
- The AzureIdentity and AzureIdentityBinding CRDs (custom resource definitions) are used to bind an identity to a pod.
- The MIC (Managed Identity Controller) and NMI (Node Managed Identity) components intercept pod requests to Azure and inject the appropriate token from the assigned identity.

### CRD Example:

```yaml
apiVersion: "aadpodidentity.k8s.io/v1"
kind: AzureIdentity
metadata:
  name: my-identity
spec:
  type: 0  # 0 = UserAssigned MSI
  resourceID: <full-azure-resource-id>
  clientID: <client-id>
---
apiVersion: "aadpodidentity.k8s.io/v1"
kind: AzureIdentityBinding
metadata:
  name: my-binding
spec:
  azureIdentity: my-identity
  selector: my-selector
```

### Pod Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  labels:
    aadpodidbinding: my-selector
spec:
  containers:
  - name: nginx
    image: nginx
```

### ⚠️ Drawbacks of Pod Identity

- Complex architecture: Requires MIC and NMI components.
- Not supported in all scenarios: Doesn't work well with CSI drivers like Secrets Store CSI.
- Deprecation: Azure has deprecated Pod Identity in favor of Workload Identity.

## ✅ What is Workload Identity?

Workload Identity is the new recommended method to authenticate pods with Azure using OIDC federation between Kubernetes Service Accounts and Azure AD.

### Advantages:

- Native to Kubernetes using OIDC.
- No need for MIC/NMI components.
- More scalable and secure.
- Works seamlessly with CSI drivers (e.g., Azure Key Vault CSI driver).

### Key Components:

- Enable OIDC Issuer during AKS cluster creation:

```bash
az aks create --enable-oidc-issuer --enable-workload-identity
```

- Federated Identity Credential on the AAD application (backed by UAMI).
- Kubernetes Service Account annotated with the client ID of the UAMI.

### Example Service Account with OIDC binding:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-service-account
  annotations:
    azure.workload.identity/client-id: <client-id>
```

### Use Case in Pod:

(Typical pod usage using the annotated service account)

### Steps
1) Create UAMI , create AzureIdentity and map UAMI details to the AzureIdentity resource.
2) Create AzureIdentityBinding and specify the AzureIdneity and Selector.
3) Specify this Selector into the Pod Labels.
