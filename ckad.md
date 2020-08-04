### CKAD

https://github.com/marcusvieira88/CKAD-commands

Set alias
```
export KUBE_EDITOR=nano
alias k=kubectl
alias kx=”kubectl explain”
```

```
kubectl run nginx --image=nginx   (deployment)
kubectl run nginx --image=nginx --restart=Never   (pod)
kubectl run nginx --image=nginx --restart=OnFailure   (job)  
kubectl run nginx --image=nginx  --restart=OnFailure --schedule="* * * * *" (cronJob)

kubectl run nginx -image=nginx --restart=Never --port=80 --namespace=myname --command --serviceaccount=mysa1 --env=HOSTNAME=local --labels=bu=finance,env=dev  --requests='cpu=100m,memory=256Mi' --limits='cpu=200m,memory=512Mi' --dry-run -o yaml - /bin/sh -c 'echo hello world'

kubectl run frontend --replicas=2 --labels=run=load-balancer-example --image=busybox  --port=8080
kubectl expose deployment frontend --type=NodePort --name=frontend-service --port=6262 --target-port=8080
kubectl set serviceaccount deployment frontend myuser
kubectl create service clusterip my-cs --tcp=5678:8080 --dry-run -o yaml
```


Replication Controller

- High Availability
- Load Balancing and Scaling

This is replaced by replica sets.


config maps

```yaml
kubectl create configmap <config-name> --from-literal=<key>=<value>
kubectl create configmap <config-name> --from-file=<path-to-file>

kubectl get configmaps
kubectl describe configmaps
```

```yaml
envFrom:
    - configMapRef:
        name: app-config

```


```

