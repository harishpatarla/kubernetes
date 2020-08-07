### CKAD

https://github.com/marcusvieira88/CKAD-commands

## Exam specific
Set alias
```
export KUBE_EDITOR=nano
alias k=$ kubectl
alias kx=”$ kubectl explain”
```

```
$ kubectl run nginx --image=nginx   (deployment)
$ kubectl run nginx --image=nginx --restart=Never   (pod)
$ kubectl run nginx --image=nginx --restart=OnFailure   (job)  
$ kubectl run nginx --image=nginx  --restart=OnFailure --schedule="* * * * *" (cronJob)

$ kubectl run nginx -image=nginx --restart=Never --port=80 --namespace=myname --command --serviceaccount=mysa1 --env=HOSTNAME=local --labels=bu=finance,env=dev  --requests='cpu=100m,memory=256Mi' --limits='cpu=200m,memory=512Mi' --dry-run -o yaml - /bin/sh -c 'echo hello world'

$ kubectl run frontend --replicas=2 --labels=run=load-balancer-example --image=busybox  --port=8080
$ kubectl expose deployment frontend --type=NodePort --name=frontend-service --port=6262 --target-port=8080
$ kubectl set serviceaccount deployment frontend myuser
$ kubectl create service clusterip my-cs --tcp=5678:8080 --dry-run -o yaml
```

```yaml
--dry-run: By default as soon as the command is run, the resource will be created. 
--dry-run=client: If you simply want to test your command, use this option. 
This will not create the resource, instead, tell you whether the resource can be created and if your command is right.
```

```bash
$ kubectl run nginx --image=nginx --dry-run=client -o yaml (Generates Pod manifest yaml file)
$ kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml (Generates Deployment YAML file)
```
kubectl create deployment does not have a --replicas option. 

You could first create it and then scale it using the kubectl scale command. You could then change replicas by updating yaml file. 

#### Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379
```bash
$ kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
or
$ kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml > redis-service.yaml
```
This will not use the pod labels as selectors, instead it will assume selectors as app=redis.

 You cannot pass in selectors as an option. So it does not work very well if your pod has a different label set. 
 So generate the file and modify the selectors before creating the service(clusterip|loadbalancer|nodeport).

#### Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes:

```yaml
$ kubectl expose pod nginx --port=80 --name nginx-service --type=NodePort --dry-run=client -o yaml 
(This will automatically use the pod's labels as selectors, but you cannot specify the node port. 
You have to generate a definition file and then add the node port in manually before creating the service with the pod).

Or

$ kubectl create svc nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml > nginx-nodeport.yaml 
(This will not use the pods labels as selectors)

I would recommend going with the `$ kubectl expose` command.
If you need to specify a node port, generate a definition file using the same command and manually input the nodeport before creating the service.
``` 
If you need to specify a node port, generate a definition file using the same command and manually input the nodeport before creating the service.

 
$ kubectl [command] [TYPE] [NAME] -o <output_format>

`-o json` Output a JSON formatted API object.

`-o name` Print only the resource name and nothing else.

`-o wide` Output in the plain-text format with any additional information.

`-o yaml` Output a YAML formatted API object. 
## Replication Controller

- High Availability
- Load Balancing and Scaling - spans across multiple nodes when load increases.

This is replaced by replica sets which is a new way to set-up replication.
 
```yaml
$ kubectl create -f rc-definition.yml
$ kubectl get replicationcontroller
$ kubectl get pods
```

### ReplicaSets

apiVersion: apps/v1 - because support for replica-set is introduced in apps/v1 version.

RS requires selector definition to define what pods fall under it because rs also manages pods which are 
not created as part of rs but match the label.

```yaml
selector:
    matchLabels:
        type: front-end
```

```yaml
$ kubectl create -f rs-definition.yml
$ kubectl get replicaset
$ kubectl get pods
$ kubectl delete replicaset myapp-replicateset (deletes underlying pods as well)
$ kubectl replace -f replicaset-definition.yml
$ kubectl scale -replicas=6 -f replicaset-definition.yml
```

RS is a process which monitors required no. of pods to be active if pods are already created and we define rs later, 
otherwise it ensures rs has required no. of pods.

Labels act as filters for RS, so RS knows which pods to monitor.

Example:
Say, 3 pods are already created. we define a RS later to monitor the pods. do we need to define the pod template again
since we already have the pods running - yes, because when one of the pod goes down this pod template will be used to create new one.

```yaml
$ kubectl scale --replicas=6 -f replicaset-definition.yml
$ kubectl scale --replicas=6 replicaset myapp-replicaset
```

## Deployment

```yaml

```

## Config Maps

```yaml
$ kubectl create configmap <config-name> --from-literal=<key>=<value>
$ kubectl create configmap <config-name> --from-file=<path-to-file>

$ kubectl get configmaps
$ kubectl describe configmaps
```

```yaml
envFrom:
    - configMapRef:
        name: app-config

```


```

