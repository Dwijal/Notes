df -h

To check the pod  disk space/file system use the below command

grep -v rootfs  /proc/mounts >  /etc/mtab


*** To copy the file from pod to local
kubectl  cp  <pod-name>:/<pod-data-file-path> /<local-file-path>

*** To copy the files from local to pod/container
kubectl cp  <filename/directoryname> <pod-name>:/pvc-webapp


#### Container logs

Location: /var/log/pods/ and /var/lib/docker/containers/<id>/*-json.log

Large images or many images = more disk used
Pulled images consume disk space in /var/lib/docker or /var/lib/containerd

df -h /var/lib/kubelet
du -sh /var/log/pods
kubectl describe pod <pod-name> | grep -A10 "Ephemeral Storage"

cat /proc/sys/kernel/pid_max           # Max allowed PIDs
ps -e | wc -l                           # PIDs currently in use

Check zombie Process 
ps -e -o stat,pid,ppid,cmd | grep '^Z'

----------------------------------------------------------------------------------

# Check pod status
kubectl get pod -n calico-system -o wide

# Check logs of a specific pod
kubectl logs calico-node-<name> -n calico-system
kubectl logs -n calico-system <calico-node-pod> | grep -Ei 'error|fail|panic|ipam|cni|vxlan|iptables|bird|BGP'

# Look for errors like:
# - failed to setup routes
# - BIRD BGP daemon crashes
# - CNI plugin failed

# Describe pod for event-level issues
kubectl describe pod calico-node-<name> -n calico-system

# Check node system logs (e.g. for containerd, network)
journalctl -u kubelet
