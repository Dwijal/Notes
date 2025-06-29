## High-Level Summary

> **CSI Controller (The Architect):** It handles all the "control plane" and non-node-specific tasks. It talks to Azure to provision the disk, creates the `PersistentVolume` (PV) object for Kubernetes to track it, and attaches that disk to the correct node VM.

> **CSI Node (The Local Worker):** It handles the "data plane" tasks on the specific node where the pod is running. Once the controller has attached the disk, the node plugin takes over to perform the final steps of formatting and mounting that disk into the requesting pod's filesystem.

---

## Azure CSI Driver: Container Cheat Sheet

This guide breaks down the containers running inside the two main CSI pods, which you can find in your AKS cluster's `kube-system` namespace.

### 1. Controller Pod (`csi-azuredisk-controller-...` or `csi-azurefile-controller-...`)

**Role:** The central brain for managing volumes across the entire cluster.

| Container Name | Its Core Responsibility |
| :--- | :--- |
| `csi-provisioner` | **Creates Azure Volumes** (Watches `PVCs` to create disks/shares) |
| `csi-attacher` | **Attaches Disks to Nodes** (Connects the Azure Disk to the node VM) |
| `csi-resizer` | **Expands Volume Size** (Handles requests to increase disk capacity) |
| `csi-snapshotter` | **Takes Volume Snapshots** (Manages backup snapshots) |
| `livenessprobe` | **Monitors Driver Health** (Checks if the driver is responsive) |
| `azuredisk` / `azurefile` | **Talks to Azure API** (The main driver executing the commands) |


### 2. Node Pod (`csi-azuredisk-node-...` or `csi-azurefile-node-...`)

**Role:** The hands-on worker running on every node to handle local volume operations.

| Container Name | Its Core Responsibility |
| :--- | :--- |
| `node-driver-registrar` | **Registers with Kubelet** (Tells the local `kubelet` it can handle mounts) |
| `livenessprobe` | **Monitors Driver Health** (Checks if the local driver is responsive) |
| `azuredisk` / `azurefile` | **Mounts Volumes Locally** (Formats, stages, and mounts the volume into the pod) |
