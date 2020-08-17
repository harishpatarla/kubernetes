### CKAD

https://github.com/marcusvieira88/CKAD-commands

https://github.com/twajr/ckad-prep-notes

https://github.com/lucassha/CKAD-resources

https://dev.to/boncheff/certified-kubernetes-application-developer-notes-10i1

https://medium.com/@harioverhere/ckad-certified-kubernetes-application-developer-my-journey-3afb0901014

https://www.linkedin.com/pulse/my-ckad-exam-experience-atharva-chauthaiwale/

https://github.com/dgkanatsios/CKAD-exercises

Auto-completion is available. You just need to take care that if you use "aliases" for referring to "kubectl" command, this auto-completion will be missed. To fix this problem you just have to follow this k8s official instructions: 
https://kubernetes.io/docs/reference/kubectl/cheatsheet/#kubectl-autocomplete

https://medium.com/@iizotov/exam-notes-ckad-c1c4f9fb9e73

Use below resources for practicing.
https://medium.com/bb-tutorials-and-thoughts/practice-enough-with-these-questions-for-the-ckad-exam-2f42d1228552
https://codeburst.io/kubernetes-ckad-weekly-challenges-overview-and-tips-7282b36a2681
https://killer.sh/ckad


## Exam specific
Set alias
```
export KUBE_EDITOR=nano
alias k=$ kubectl
alias kx=”$ kubectl explain”
export ns=default
alias kn='kubectl -n $ns' # This helps when namespace in question doesn't have a friendly name 
alias kdr= 'kubectl -n $ns -o yaml --dry-run'.  # run commands in dry run mode and generate yaml.
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

More alias for exam:

```bash
sudo -i
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc
complete -F __start_kubectl k
export do="--dry-run=client -o yaml"
export now="--force --grace-period=0"
alias k="kubectl"
alias kcf="k create"
alias kdf="k delete $now"
alias kdff="kdf -f"
alias krp="k run"                   
alias krpy="krp $do"
alias krd="k create deployment"
alias krdy="krd $do"
If you simply want to create a Pod use :
krp --image=nging nginx
If you want pod with yaml use:
krpy --image=nging nginx > nginx-pod.yaml
If you simply want to create a Deployment use :
krd --image=nging nginx
If you want deployment with yaml use:
krdy --image=nging nginx > nginx-dep.yaml
If you want to delete a Pod use :
kdf po nginx
If you want to delete a Pod for which yaml is present use :
kdff nginx-pod.yaml
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

* #### Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379
```bash
$ kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
or
$ kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml > redis-service.yaml
```
This will not use the pod labels as selectors, instead it will assume selectors as app=redis.

 You cannot pass in selectors as an option. So it does not work very well if your pod has a different label set. 
 So generate the file and modify the selectors before creating the service(clusterip|loadbalancer|nodeport).

* #### Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes:

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

* #### Replication Controller

- High Availability
- Load Balancing and Scaling - spans across multiple nodes when load increases.

This is replaced by replica sets which is a new way to set-up replication.
 
```yaml
$ kubectl create -f rc-definition.yml
$ kubectl get replicationcontroller
$ kubectl get pods
```

* #### ReplicaSets

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

* #### Namespace

```yaml
kubectl create -f namespace-dev.yaml
kubectl create namespace dev
kubectl get pods --namespace=dev
kubectl config set-context $(kubectl config current-context) --namespace=dev (switch to dev namespace)
kubectl get pods --all-namespaces

```
### Section 3: Configuration

* #### docker 
```yaml
docker run ubuntu
docker ps
docker ps -a
```
container exists as long as process inside it is alive. Once the process is done container will exit.

* #### commands and arguments in k8s
* ##### Edit a pod - 1st approach

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

* ##### Edit a pod - 2nd approach
```yaml
kubectl get pod webapp -o yaml > my-new-pod.yaml
vi my-new-pod.yaml
kubectl delete pod webapp
kubectl create -f my-new-pod.yaml
```

* ##### Edit Deployments
With Deployments you can easily edit any field/property of the POD template.
 
Since the pod template is a child of the deployment specification,  with every change the deployment will automatically delete and create a new pod with the new changes. 

So if you are asked to edit a property of a POD part of a deployment you may do that simply by running the command.

`kubectl edit deployment my-deployment`

* #### Def Env. variables in pod defination file 

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

* #### Docker Security
```yaml
docker run --cap-add MAC_ADMIN ubuntu 
```

* #### Security context 
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

* #### Service accounts

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

* #### Resource Requirements
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

* #### Taints and Tolerations

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

* #### Node Selector

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

* #### Node Affinity

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

### Section 4: Multi-container pods

you sometimes need pods to work together with some level of seperation of concerns.

However, they would share the same  
    
    - lifecycle - created and destroyed together
    - network - they can refer each other as localhost
    - have access to same storage volume 
    
1.  _Sidecar_ - `Log server container as sidecar container along with the main container`
    
2.  _Adapter_ - `Formatting logs generated in different formats to a common format before sending to log server`

3. _Ambassador_ - `Application may need to talk to different DB env during diff. stages of development.
 We can extract that logic to another component and Application can always talk to localhost 
 and that will proxy the request to specific env`

### Section 5: Observability

* #### Readiness and Liveness probes

_Status:_ `Pending -> ContainerCreating -> Running`

_Conditions:_ `PodScheduled -> Initialized -> ContainersReady -> Ready`

```yaml
spec:
    containers:
      - name : simple-webapp
        readinessProbe:
          httpGet:
            path: /api/ready
            port: 8080
          tcpSocket:
            port: 3306
          exec:
            command:
              - cat
              - /app/is_ready
        initialDelaySeconds: 10
        periodSeconds: 5
        failureThreshold: 8
```
* #### Container logging

```bash
$ kubectl logs -f pod-name container-name
```

* #### Monitor and Debug Applications

-   Heapster(deprecated) - metrics server
-   Prometheus
-   Datadog

Both of these gives 
```bash
$ kubectl top node
$ kubectl top pod
```

### Section 6 - Pod Design

* #### Labels, Selectors and Annotations
3 pods will be labeled as _app: App1_ and replica set will have selector: matchLabels - app: App1 to link 3 pods to the replica set.
```bash
$ kubectl get pods --selector app=App1
```

_Annotations_ are used to save other info like buildversion, email, contact etc.

* #### Rolling updates and Rollbacks in Deployments

```bash
$ kubectl create -f deployment-definition.yml - CREATE
$ kubectl get deployments - GET
$ kubectl apply -f deployment-definition.yml - UPDATE
$ kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1 - UPDATE
$ kubectl rollout status deployment/myapp-deployment - STATUS
$ kubectl rollout history deployment/myapp-deployment - STATUS
$ kubectl rollout undo deployment/myapp-deployment - ROLLBACK
```

* Default deployment strategy - _RollingUpdate_

You can check the status of each revision individually by using the --revision flag

```bash
$ kubectl rollout history deployment nginx --revision=1
```

```bash
$ kubectl set image deployment nginx nginx=nginx:1.17 --record=true
$ kubectl describe deployments. nginx | grep -i image:
  Image:    nginx:1.17
```

* #### 
