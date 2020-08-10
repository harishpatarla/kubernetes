### CKAD

https://github.com/marcusvieira88/CKAD-commands

https://github.com/twajr/ckad-prep-notes

https://dev.to/boncheff/certified-kubernetes-application-developer-notes-10i1

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

### Replication Controller

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

#### Namespace

```yaml
kubectl create -f namespace-dev.yaml
kubectl create namespace dev
kubectl get pods --namespace=dev
kubectl config set-context $(kubectl config current-context) --namespace=dev (switch to dev namespace)
kubectl get pods --all-namespaces

```

#### docker 
```yaml
docker run ubuntu
docker ps
docker ps -a
```
container exists as long as process inside it is alive. Once the process is done container will exit.

#### commands and arguments in k8s
##### Edit a pod - 1st approach

```yaml
`kubectl edit pod <pod name>` - maybe denied
kubectl delete pod webapp
A temp file will be created and you can use that to create a pod
kubectl create -f /tmp/kubectl-edit-ccvrq.yaml 
```
Remember, you CANNOT edit specifications of an existing POD other than the below.

1.  spec.containers[*].image
2.  spec.initContainers[*].image
3.  spec.activeDeadlineSeconds
4.  spec.tolerations

##### Edit a pod - 2nd approach
```yaml
kubectl get pod webapp -o yaml > my-new-pod.yaml
vi my-new-pod.yaml
kubectl delete pod webapp
kubectl create -f my-new-pod.yaml
```

##### Edit Deployments
With Deployments you can easily edit any field/property of the POD template.
 
Since the pod template is a child of the deployment specification,  with every change the deployment will automatically delete and create a new pod with the new changes. 

So if you are asked to edit a property of a POD part of a deployment you may do that simply by running the command.

`kubectl edit deployment my-deployment`

#### Def Env. variables in pod defination file 

1.  Plain key value
```yaml
env:
  - name: APP_COLOR
    value: pink
```
2.  Config Map
```yaml
env:
  - name: APP_COLOR
    valueFrom:
      configMapKeyRef:
``` 
3.  Secrets
```yaml
env:
  - name: APP_COLOR
    valueFrom:
      secretKeyRef:
```
### Config Maps

```yaml
$ kubectl create configmap <config-name> --from-literal=<key>=<value>
$ kubectl create configmap <config-name> --from-file=<path-to-file>

$ kubectl create configmap app-config --from-literal=APP_COLOR=BLUE \
                                      --from-literal=APP_ENV=PROD
$ kubectl create configmap <config-map> --from-file=app_config.properties

$ kubectl get configmaps
$ kubectl describe configmaps
```

1.  Env
```yaml
envFrom:
    - configMapRef:
        name: app-config
```
2. Single Env
```yaml
env:
    - name: APP_COLOR
      valueFrom:
        configMapKeyRef:
          name: app-config
          key:  APP_COLOR
``` 

3.  Volume
```yaml
volumes:
    - name: app-config-volume
      configMap: 
        name: app-config
```

### Secrets

```yaml
kubectl create secret generic <secret-name> --from-literal=<key> = <value>
kubectl create secret generic app-secret --from-literal=DB_HOST=mysql

kubectl create secret generic <secret-name> --from-file= <path-to-file>
kubectl create secret generic app-secret --from-file=app_secret.properties

kubectl get secrets
kubectl describe secrets
kubectl get secret app-secret -o yaml
```

`echo -n 'abcdef' | base64 --decode'`


1.  Env
```yaml
envFrom:
    - secretRef:
        name: app-secret
```
2. Single Env
```yaml
env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secret
          key:  DB_PASSWORD
``` 

3.  Volume
```yaml
volumes:
    - name: app-secret-volume
      secret: 
        secretName: app-secret
```

Secrets encode data in base64 format. Anyone with the base64 encoded secret can easily decode it. 
As such the secrets can be considered as not very safe.

Secrets are not encrypted, some best practices around using secrets make it safer.

1.  Not checking-in secret object definition files to source code repositories.
2.  Enabling Encryption at Rest for Secrets, so they are stored encrypted in ETCD. 

Also the way kubernetes handles secrets. Such as:

1.  A secret is only sent to a node if a pod on that node requires it.
2.  Kubelet stores the secret into a tmpfs so that the secret is not written to disk storage.
3.  Once the Pod that depends on the secret is deleted, kubelet will delete its local copy of the secret data as well.

Risks - https://kubernetes.io/docs/concepts/configuration/secret/#risks
Protection - https://kubernetes.io/docs/concepts/configuration/secret/#protections

There are other better ways of handling sensitive data like passwords in Kubernetes, such as using tools like Helm Secrets, HashiCorp Vault.

#### Docker Security
```yaml
docker run --cap-add MAC_ADMIN ubuntu 
```

#### Security context 
```yaml
spec:
    containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "3600"]
      securityContext: 
        runAsUser: 1000
        capabilities: 
          add: ["MAC_ADMIN"]
```
Capabilities are only supported at the container level and not at the pod level.

#### Service accounts

User accounts are used my humans - admin to manage cluster, dev to deploy an application 

service accounts are used by applications - Prometheus, Jenkins etc. 
Prometheus pools the k8s apis to extract.

```yaml
$ kubectl create serviceaccount dashboard-sa
$ kubectl get serviceaccount
$ kubectl describe serviceaccount dashboard-sa
$ curl https://k8s-master/api -insecure --header "Authorization: Bearer ..."

$ kubectl exec -it my-k8s-dahboard ls /var/run/secrets/k8s.io/serviceaccount
$ kubectl exec -it my-k8s-dahboard ls /var/run/secrets/k8s.io/serviceaccount/token

``` 

#### Resource Requirements
K8s scheduler decides which node a pod goes to

min. resource

```yaml
resources:
    requests:
      memory: "1Gi"
      cpu: 1
    limits:
      
```

1 CPU = 1 AWS vCPU, 1 GCP core, 1 Azure core, 1 hyper-thread

1KB = 1000 bytes
1MB = 1000 KB
1GB = 1000 MB

1 GiB = 1024 MiB
1 MiB = 1024 KiB
1 KiB = 1024 bytes

when pod tries to exceed resources, k8s throttles the CPU so that it does not go beyond the specified limit.
However, this is not the case with the memory, a container can use more memory resources than its limit.
So if a pod tries to consume more memory than its limit constantly the pod will be terminated.


When a pod is created the containers are assigned a default CPU request of .5 and memory of 256Mi".
For the POD to pick up those defaults you must have first set those as default values for request and limit by creating a LimitRange in that namespace.

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
spec:
  limits:
  - default:
      memory: 512Mi
    defaultRequest:
      memory: 256Mi
    type: Container
```

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-limit-range
spec:
  limits:
  - default:
      cpu: "1"
    defaultRequest:
      cpu: "0.5"
    type: Container
```

#### Taints and Tolerations

Taints are set on nodes and tolerations are set on pods.

Taints and Tolerations does not tell a pod to go on particular node. Instead it only tells node to accept a pod with 
certain tolerations.

```yaml
$ kubectl taint nodes node-name key=value:taint-effect
```
taint-effect - NoSchedule | PreferNoSchedule | NoExecute

```yaml
$ kubectl taint nodes node1 app=blue:NoSchedule
```

```yaml
spec:
    tolerations:
      - key: "app"
        operator: "Equal"
        value: "blue"
        effect: "NoSchedule"
```

#### Node Selector

```yaml
spec:
    nodeSelector:
      size: Large #label on the node
```

Label Nodes 

`kubectl label nodes <node-name> <label-key> = <label-value>`

`kubectl label nodes node-1 size=large`

Limitations of Node selector 
    - Large OR Medium?
    - NOT Small? 

For these we have to use Node Affinity

#### Node Affinity

```yaml
affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: size
                operator: In
                values:
                  - Large
                  - Medium              
              - key: size
                operator: NotIn
                values:
                  - Small
```

What if there are no nodes with label size?

Node Affinity Types:

Available

    - requiredDuringSchedulingIgnoredDuringExecution
    - preferredDuringSchedulingIgnoredDuringExecution
    
Planned

    - requiredDuringSchedulingRequiredDuringExecution
    

