### Sec 22: Horizontal Pod Autoscaling(HPA)

Every pod can be replicated but you need to make sure that that software is designed
so that if it's replicated, the system is going to continue to behave properly.

you need to make sure that that software is designed
so that if it's replicated, the system is
going to continue to behave properly.

So, in a Kubernetes cluster, when we take a pod
and we create multiple instances of that pod,
we are horizontally scaling.

We can automatically autoscale at runtime.
Make sure that the metrics-server is enabled for hpa. 
Earlier heapster was used but is now deprecated

Lets say our CPU request is **_100 mi_**
We can define a hpa rule:

`If api-gateway > 50 % CPU then
Autoscale to a maximum of 4 pods.
`

`If api-gateway < 50 % CPU then Autoscale will scale down.`

```
kubectl autoscale deployment api-gateway --cpu-percent 400 --min 1 --max 4
kubectl get hpa api-gateway -o yaml
kubectl describe hpa api-gateway
kubectl delete hpa api-gateway
```

```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: api-gateway
  namespace: default
spec:
  maxReplicas: 4
  minReplicas: 1
  scaleTargetRef:
    apiVersion: extensions/v1beta1
    kind: Deployment
    name: api-gateway
  targetCPUUtilizationPercentage: 400
```

After an autoscale there is a wait time as the CPU is monitored and only if it consistently is below threshold will it scale down again.
This is to avoid scale up and down events to happen too frequently.

### Sec 23: Readiness and Liveness
##### Why we need Readiness probe

There will be a period when the pod's running but it's not able to do anything useful
because the programme inside the pod is still starting up.

When we have a readiness probe,Kubernetes will not send traffic to that pod
until the readiness probe is showing that the pod is ready to receive requests.

With this status of new pods will be running but READY - 0/1.

##### Why we need Liveness probe
When we have some issue with pod then this pod will not be live and will not get any requests.

### Section 24: Quality of Service and Eviction
![alt text](https://github.com/harishpatarla/kubernetes/blob/master/RichCourses/QualityOfService.png)

1. **Guaranteed**: Pod with a request and a limit,and those values are the same
2. **Burstable**: If you have a pod which specifies a request but no limit,
and this could be a memory request and/or a CPU request.
_Pod is allowed to go over its requested memory/CPU. Request is just a hint to how much pod can use to run comfortably. So pod can burst._
3. **BestEffort**: If you have a pod which neither specifies request nor limit.

These labels are used by the scheduler when node runs out of resources.

These labels will help the scheduler decide which pods to evict if the node is under pressure.

Linux cgroups are used to evict the pods.
It will evict any Best Effort pods first.
then it will start to evict the Burstable pods.
Only then, if it really can't free up
enough resources from that,
will it start to evict Guaranteed pods.

If you have more than one container running in 
a pod then you need to set req/limits on both containers
 for it to get a desired label.
### Section 25: RBAC (Role Based Access Control)

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
   name: new-joiner
   namespace: default
rules:
- apiGroups: ["","apps"] # Core API(for services) AND apps(for deployment)
  resources: ["*"]  # pods, services, deployments...maybe not for secrets
  verbs: ["get", "list", "watch"]

```
any new joiner will only be able to do get, list and watch.
_**apiGroups**_ 
_**"watch"**_ calls the rest API on the cluster and subscribe to changes.
Spring cloud kubernetes uses this to watch for changes made to ConfigMaps.

##### Defining RoleBindings 
Allows us to put users into new-joiner role

```yaml

kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: put-specific-user-or-users-into-new-joiner-role
  namespace: default
subjects:
- kind: User
  name: francis-linux-login-name
roleRef:
  kind: Role
  name: new-joiner
  apiGroup: rbac.authorization.k8s.io

```
`kubectl config view` will give below output

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://harish.k8s.local:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    namespace: harish-playground
    user: harishpatarla
  name: harish@kubernetes
current-context: harish@kubernetes
kind: Config
preferences: {}
users:
- name: harishpatarla
  user:
    auth-provider:
```

new user can get access to cluster in secured way
kubectl config set-context mycontext --user harish --cluster kubernetes

If you want to set multiple contexts for multiple clusters follow below:
https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/

x509 certificate should be given to new-joiner user by the root user of the cluster.

There would be certificate authority hidden in the cluster and only the root user can access it.
root user can generate private key to give to the new joiner

`openssl genrsa -out private-key-new-joiner.key 2048`

now we need a certificate signing request 
```
openssl req -new -key private-key-new-joiner.key -out req.csr -subj "/CN=francis-linux-login-name/0=francis-linux-login-name"
```

These Roles and RoleBindings are specific to a namespace.
 
To make them apply to whole of cluster, use  ClusterRole and ClusterRoleBinding

### Section 26: Kubernetes ConfigMaps and Secrets

Feed values of 


### sec 27 - Ingress controller
So far we used NodePort - to access service on a specific port.
OR resource of type clusterIP which can be accessed from only within a cluster.

This is absolutely fine if you have one service
that you want to publish to your end users.
But if you have more than one service you would need one LB for each service.

The advantage to this is the load balancers are completely separate hardware entities,
and therefore will have different IP addresses/different domain names, you can have them both listening to port 80.

You can use ALB too where you can set routing rules.

But k8s does not understand ALB(AWS) as k8s is cloud agnostic
We can instead use Ingress controller(Nginx). 

Based on the domain name it can route to different services - Fanning out

---
Now for minikube we can just enable ingress using addons
Inrgess routes requests to default http backend

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: basic-routing
spec:
  rules:
    - host: fleetman.com
      http:
        paths:
          - path: /
            backend:
              serviceName: fleetman-webapp
              servicePort: 80
    - host: queue.fleetman.com
          http:
            paths:
              - path: /
                backend:
                  serviceName: fleetman-queue
                  servicePort: 8161
```
If we apply security to the queue as it is only for staff within the company,
our fleetman.com will also become secure

So we can seperate these out in 2 files.
fleetman.com will be unsecured.

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: public-routing
spec:
  rules:
    - host: fleetman.com
      http:
        paths:
          - path: /
            backend:
              serviceName: fleetman-webapp
              servicePort: 80
```

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: secure-routing
  annotations:
    nginx.ingress.kubernetes.io/auth-type: basic
    nginx.ingress.kubernetes.io/auth-secret: mycredentials
    nginx.ingress.kubernetes.io/auth-realm: "Get lost unless you have a password"
spec:
  rules:
    - host: queue.fleetman.com
      http:
        paths:
          - path: /
            backend:
              serviceName: fleetman-queue
              servicePort: 8161
```
The pre-requisite generic deployment command,
 


### sec 28 - Other workloads 
Batch Jobs  

```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: cron-job
spec: # CronJob
  schedule: "* * * * *"
  jobTemplate:
    spec: # JOB
      template:
        spec: # Pod
          containers:
          - name: long-job
            image: python:rc-slim
            command: ["python"]
            args: ["-c", "import time; print('starting'); time.sleep(30); print('done')"]
          restartPolicy: Never
      backoffLimit: 2
```
#### Daemon Sets

A DaemonSet ensures that all (or some) Nodes run a copy of a Pod. 
As nodes are added to the cluster, Pods are added to them. 
As nodes are removed from the cluster, those Pods are garbage collected. 
Deleting a DaemonSet will clean up the Pods it created.

For deamon set there is no replicas, but otherwise it is same as deployment syntax.

#### Stateful Sets
They are not used for persistence. Not related to PV, PVC and EBS

Usually we don't care of the which replica of the pods the request is served by. 
Pods are treated as cattle not pets

Sometimes you need a set of pods with known, predictable names
And you want clients to be able to call them by their name.

If for specific reason client needs to call pod1 directly, 
we can use PetSet.

##### PetSet:
1. The Pods will always start up in sequence. pod2 starts only after pod1 is up and running
2. This would allow a client to address the pods individually
by name rather than just being serviced by random pods.
3. Pods will have predictable names.
 
Client does not call directly, instead Headless service can be used.
Each pod gets a copy of this service.

_Headless Service is just same as Service but the fact that it connects to a StatefulSet makes it  
Headless Service._

When you need replication of mongoDB or other kinds of database, we can't just have replicas:3
It would give 3 independent mongo DBs and there is no sync
StatefulSets solve this problem.

Mongo has election where 1 instance will be primary and secondary instances will be replicated.
All writes should always go to primary.You can't do writes to the secondaries.

The URL for a headless service is, 
<the name of the pod that you want to address>.<followed by the name of the service>

Eg. mongo-0.mongodb, mongo-1.mongodb, mongo-2.mongodb/fleetman

So when client writes to the Mongo cluster,it will make a call to the URL and primary pod serves it.
Client's not going to know which are the primaries and the secondaries.

So the MongoDB URL is a comma-separated list of all the Mongo pods.
And the client needs to know the names of these pods.
If anything happened to mongo-0,a primary would be elected from the other 2 pods.

_StatefulSets are useful to **replicate and scale out databases.**_ 

```yaml
# Demo of a STATEFUL set, Mongo clients need to be able to address the pods
# by name, so a statefulset and headless service are used.
# Clients will connect using eg mongodb://mongo-0.mongo,mongo-1.mongo,mongo-2.mongo/fleetman

apiVersion: v1
kind: Service
metadata:
 name: mongo
 labels:
   name: mongo
spec:
 ports:
 - port: 27017
   targetPort: 27017
 clusterIP: None
 selector:
   role: mongo
---
apiVersion: apps/v1beta1
kind: StatefulSet
metadata:
 name: mongo
spec:
 replicas: 3
 serviceName: "mongo"
 template:
   metadata:
     labels:
        role: mongo
   spec:
     terminationGracePeriodSeconds: 10
     containers:
       - name: mongo
         image: mongo:3.6.5-jessie
         command:
           - mongod
           - "--replSet"
           - rs0
           - "--smallfiles"
           - "--noprealloc"
           - "--bind_ip"
           - "0.0.0.0"
         ports:
           - containerPort: 27017
         volumeMounts:
           - name: mongo-persistent-storage
             mountPath: /data/db
       - name: mongo-sidecar
         image: cvallance/mongo-k8s-sidecar
         env:
           - name: MONGO_SIDECAR_POD_LABELS
             value: "role=mongo"
           - name: KUBERNETES_MONGO_SERVICE_NAME
             value: mongo
 volumeClaimTemplates:
  - metadata:
      name: mongo-persistent-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: "standard"
      resources:
        requests:
          storage: 7Gi
---
# See https://github.com/cvallance/mongo-k8s-sidecar/issues/75
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: default-view
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: view
subjects:
  - kind: ServiceAccount
    name: default
    namespace: default

```

