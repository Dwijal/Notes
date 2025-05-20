
## 📦 Kubernetes Environment Variables and Downward API from secrets , configmap and volume Mounts.
---

## 1. Static Environment Variable

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: static-env-pod
spec:
  containers:
    - name: app
      image: nginx
      env:
        - name: APP_MODE
          value: "production"
```

---

## 2. Pass ENV variable From ConfigMap (Single Key)

### ConfigMap:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  DB_HOST: mysql.default.svc.cluster.local
```

### Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-env-pod
spec:
  containers:
    - name: app
      image: nginx
      env:
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: DB_HOST
```

---

## 3. From ENV variables from ConfigMap (All Keys)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-envfrom-pod
spec:
  containers:
    - name: app
      image: nginx
      envFrom:
        - configMapRef:
            name: app-config
```

---

## 4. From ENV variables from Secret (Single Key)

### Secret:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  DB_PASSWORD: cGFzc3dvcmQ=  # base64 for "password"
```

### Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
    - name: app
      image: nginx
      env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: DB_PASSWORD
```

---

## 5. Pass ENV variables From Secret (All Keys)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-envfrom-pod
spec:
  containers:
    - name: app
      image: nginx
      envFrom:
        - secretRef:
            name: db-secret
```

---

## 6. Downward API via Environment Variables

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: downwardapi-env-pod
spec:
  containers:
    - name: app
      image: busybox
      command: ["sh", "-c", "env && sleep 3600"]
      env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
```

---

## 7. Downward API via Volume

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: downwardapi-volume-pod
  labels:
    app: myapp
spec:
  containers:
    - name: app
      image: busybox
      command: ["sh", "-c", "cat /etc/podinfo/labels && sleep 3600"]
      volumeMounts:
        - name: podinfo
          mountPath: /etc/podinfo
          readOnly: true
  volumes:
    - name: podinfo
      downwardAPI:
        items:
          - path: "labels"
            fieldRef:
              fieldPath: metadata.labels
```

---

## 8. Mount Secret as Volume

### Secret:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  username: YWRtaW4=        # base64 for "admin"
  password: cGFzc3dvcmQ=    # base64 for "password"
```

### Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-volume-pod
spec:
  containers:
    - name: app
      image: busybox
      command: ["sleep", "3600"]
      volumeMounts:
        - name: secret-volume
          mountPath: "/etc/secrets"
          readOnly: true
  volumes:
    - name: secret-volume
      secret:
        secretName: app-secret
```

> You can now read files inside `/etc/secrets/username` and `/etc/secrets/password`.

---


## 🔧 Step 1: Create the ConfigMap with Multiple Keys

Each key will become a file, and the value will be the content of that file.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config-volume
data:
  app.properties: |
    mode=production
    timeout=30
  db.properties: |
    host=mysql.default.svc.cluster.local
    port=3306
  logging.conf: |
    level=DEBUG
    file=/var/log/app.log
```

---

## 📦 Step 2: Mount the ConfigMap into a Pod as a Volume

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-volume-pod
spec:
  containers:
    - name: app
      image: busybox
      command: ["sh", "-c", "ls /etc/config && sleep 3600"]
      volumeMounts:
        - name: config-volume
          mountPath: /etc/config
          readOnly: true
  volumes:
    - name: config-volume
      configMap:
        name: app-config-volume
```

---

## 📁 Result Inside the Pod

The following files will appear under `/etc/config/`:

```
/etc/config/app.properties
/etc/config/db.properties
/etc/config/logging.conf
```

Your app can then read them just like normal configuration files.

---

## 🔄 Dynamic Reload Support

If your app supports watching files for changes (e.g., via inotify or polling), it can reload config without restarting the pod. Kubernetes will automatically update mounted config files when the ConfigMap changes.

---
