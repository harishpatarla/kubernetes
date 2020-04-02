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

