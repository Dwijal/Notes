### - WITH block defines the scope of a particular object or  a variable .
### - If we have multiple nested values in values.yaml file then we can use with block here .

```
{{ with PIPELINE }}
  # restricted scope
{{ end }}
```

EG.

```yaml
# For testing Flow Control: with
# sample values.yaml file
podAnnotations: 
  appName: myapp1
  appType: webserver
  appTech: HTML
```
- `with` action statement sets the dot obejct "." to `.Values.podAnnotations` 
- Inside the `with` action block dot "." always refers to `.Values.podAnnotations` 
- Outside the `with` action block dot "." refers to Root Object
```yaml
  template:
    metadata:
      {{- with .Values.podAnnotations }}
      annotations:
        {{- toYaml . | nindent 8 }}        
      {{- end }}    
```
- To access Root Objects inside `with` action block we need to prepend that Root object with `$`
```yaml
# Add Root Object in with Block
  template:
    metadata:
      {{- with .Values.podAnnotations }}
      annotations:
        {{- toYaml . | nindent 8 }}
        appManagedBy: {{ $.Release.Service }}
      {{- end }}
```
- How to retrieve a single object from `.Values.myapps.data.config` ?
- What if there is only need for 1 or 2 values from `.Values.myapps.data.config` ?
- How to access each key value from `.Values.myapps.data.config` ?
```yaml
# values.yaml
# For testing Flow Control: with - Scope more detailed
myapps:
  data: 
    config: 
      appName: myapp1
      appType: webserver
      appTech: HTML
      appDb: mysql

# Current Scope: Retrieve single object using scope
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}
data: 
{{- with .Values.myapps.data.config }}
  application-name: {{ .appName }}
  application-type: {{ .appType }}
{{- end}}
```

### Helm Development - Flow Control If-Else
### IF-ELSE Syntax
```t
{{ if PIPELINE }}
  # Do something
{{ else if OTHER PIPELINE }}
  # Do something else
{{ else }}
  # Default case
{{ end }}
```

## Review values.yaml
```yaml
# If, else if, else
myapp:
  env: prod
  retail:
    enableFeature: true
```

## Step-03: Logic and Flow Control Function: and 
- [Logic and Flow Control Functions](https://helm.sh/docs/chart_template_guide/function_list/#logic-and-flow-control-functions)
- **and:**  Returns the boolean AND of two or more arguments (the first empty argument, or the last argument).
```t
# and Syntax
and .Arg1 .Arg2
```
## Step-04: Implement if-else for replicas with Boolean 

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}
  labels:
    app: nginx
spec:
{{- with .Values.myapp }}
{{- if and .retail.enableFeature (eq .env "prod") }}
  replicas: 6
{{- else if eq .env "prod" }}
  replicas: 4
{{- else if eq .env "qa" }}  
  replicas: 2
{{- else }}  
  replicas: 1  
{{- end }}
{{- end }}
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: ghcr.io/stacksimplify/kubenginx:4.0.0
        ports:
        - containerPort: 80
```
