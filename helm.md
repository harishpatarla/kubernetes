server side component - tiller - pod running in the cluster

helm is package installer - packages are called charts in helm world

own helm chart and repository 

for repetative tasks - helm

helm to deploy chart in standard way
no typo, documentation would be there

If you want to use a chart but override some properties you can do that.

Helm looks at kube/config file and knows which cluster and namespace etc

RBAC to be used 

we need to give Tiller permissions to make deployments to namespace

We need to ClusterRoleBinding for Tiller service account  so we can do deployments

```
kubectl -n kube-system create serviceaccount tiller
kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
kubectl get clusterrolebinding tiller
helm init --service-account tiller # This creates tiller pod in k8s cluster
helm home # home directory for helm
helm list
helm inspect stable/jenkins #This command inspects a chart and displays information. It takes a chart reference('stable/drupal'), a full path to a directory or packaged chart, or a URL.

kubectl -n kube-system get deploy,pod,replicaset,clusterrolebinding,serviceaccount | grep tiller

#uninstalls helm (-f to remove tiller too)
helm reset (will not remove home directory)
helm reset --remove-helm-home
kubectl get po,deploy,rs,svc | grep kafka
helm version --short --client
```

#### Configuring Helm Security
service account
Tiller runs as pod with previledge of service account
tiller has cluster admin rights
In PROD restrict tiller access on k8s cluster

Add tiller service account

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
    
