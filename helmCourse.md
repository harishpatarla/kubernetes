Below is repo used for this course of helm
https://github.com/phcollignon/helm
#### Configuring Helm Security
service account
Tiller runs as pod with privilege of service account.

Tiller has cluster admin rights.

In PROD restrict tiller access on k8s cluster.

Add Below tiller service account:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
 name: tiller
 namespace: lab

```

Add tiller role and role binding
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
 name: tiller-role
 namespace: lab
rules:
- apiGroups: ["", "extensions", "apps"]
  resources: ["*"]
  verbs: ["*"]
- apiGroups: ["batch"]
  resources:
  - jobs
  - cronjobs
  verbs: ["*"]

```
Tiller RoleBinding
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
 name: tiller-role-binding
 namespace: lab
subjects:
- kind: ServiceAccount
  name: tiller
  namespace: lab
roleRef:
  kind: Role
  name: tiller-role 
  apiGroup: rbac.authorization.k8s.io

```

```
kubectl create namespace lab
kubectl create -f tiller-serviceaccount.yaml
kubectl create -f tiller-role.yaml
kubectl create -f tiller-role-binding.yaml
helm init --service-account tiller --tiller-namespace lab

```
With this tiller is allowed to install charts only on lab namespace.
It is not allowed to install charts on default namespace - gives forbidden error.
Install tiller locally, helm and tiller would still communicate using gRPC.

for helm to be able to find tiller 
```
export HELM_HOST=localhost:44134
``` 
Helm 3 has no tiller component

#### Build Helm chart

chart properties in chart.yaml

Template folder contains - yaml for k8s workloads - deploy,svc,po,ingress,rbac,secrets,configmaps etc.

They are called templates because it has placeholders which can be replaces by values in values.yaml

Tests folder contains pod definitions for testing

Chart is **definition** of application.

Release is **instance** of that chart in k8s.

Application version
Chart version

Difference releases specific to env can be installed in different env specific clusters

New revision of same Release - Release revision

1. Installing a chart
    
    ```
   helm install guestbook
   
    ```
    
2.  Upgrading a release
    
    change to appVersion: 1.1
   ```
   helm upgrade <name of release> <name of the chart>
   helm upgrade modest-pike guestbook
   ``` 
    
3. Rolling back a release
    ```
       helm rollback <name of release> <revision>
       helm rollback modest-pike 1
       helm history modest-pike
    ``` 
       
4. Deleting a release    
    ```
       helm delete modest-pike --purge
    ``` 
 #### Building an umbrella helm
![alt text](https://github.com/harishpatarla/kubernetes/blob/master/images/helm1.png)

Helm Template help us reuse k8s workloads we create

Helm Template Engine:
![alt text](https://github.com/harishpatarla/kubernetes/blob/master/images/helmTemplate.png)

helm get <release_name>

#### Playing with helm template data
values.yaml can have values

values can also be in other-file.yaml but need to be referenced as -f file

`{{.Values.service.name}}`

Chart data can be accessed from chart.yaml 

`
{{.Chart.Name}}
`

Release data and Kubernetes data 

`{{.Capabilities.KubeVersion}}
`

Template data 

Helm do not support - but _ in values.yaml. Ex: guestbook_name: {{.Chart.Name}}

```
helm lint
```

#### Functions and pipelines

quote(value)
default(default_value, value)

quote | value
value | default default_value
value | upper | quote


![alt text](https://github.com/harishpatarla/kubernetes/blob/master/images/helm2.png)

Functions and pipelines come from 
Go text/template package,
sprig functions,
helm project.

![alt text](https://github.com/harishpatarla/kubernetes/blob/master/images/helmfunctionsandpipelines.png)

Modifying scope - _"with"_ 

![alt text](https://github.com/harishpatarla/kubernetes/blob/master/images/removingcarriagereturnfromdefinitions.png)

#### Helper functions


_helpers.tpl
```yaml
{{- define "backend.fullname" -}}
{{- if .Values.fullnameOverride -}}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" -}}
{{- else -}}
{{- printf "%s-%s" .Release.Name .Chart.Name | trunc 63 | trimSuffix "-" -}}
{{- end -}}
{{- end -}}
```
These are called sub-templates.

.helmignore - add *.tpl to it

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: {{ include "backend.fullname" . }}-secret 
data:
  mongodb-uri: {{ with .Values.secret.mongodb_uri -}}
  {{- list "mongodb://" .username ":" .password "@" $.Release.Name "-" .dbchart ":" .port "/" .dbconn | join ""  | b64enc |  quote }}
# {{- ( printf "%s%s:%s@%s-%s%s" "mongodb://" .username .password $.Release.Name "database" ":27017/guestbook?authSource=admin" ) | b64enc | quote }}
{{- end }}
```

### Managing dependencies

packaging a chart  in a .tar.gz archive
add version to the archive

`helm package chart-name`

publishing in a repository

![alt text](https://github.com/harishpatarla/kubernetes/blob/master/images/helmrepository.png)
