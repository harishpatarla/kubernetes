# k8s with kind if you only have docker env to work with.

If you only have docker env to work with then minikube is not an option for you.
If you want k8s in docker vm then kind(k8s in docker) can be used.
After installing Kind and creating the kind cluster we might want to quickly start spinning workloads in k8s cluster. 
For that we need docker images if we are not building one.

We need to do the below steps which comes from [kind](https://kind.sigs.k8s.io/docs/user/quick-start/) 
quick start guide:

```
docker build -t my-custom-image:unique-tag ./my-image-dir
kind load docker-image my-custom-image:unique-tag
kubectl apply -f my-manifest-using-my-image:unique-tag
```

If you want to pull an image from docker registry and spin up a workload then use below steps: 
```
docker pull my-custom-image:unique-tag
kind load docker-image my-custom-image:unique-tag
kubectl apply -f my-manifest-using-my-image:unique-tag
```

Below are steps to run sample image kubia from k8s in action book:
```
docker pull luksa/kubia:v3
kind load docker-image luksa/kubia:v3
kubectl create deploy kubia --image=luksa/kubia:v3
```
Here is what you should see if all goes well

```
DEV [ ~]$ docker pull luksa/kubia:v3
Trying to pull repository docker.io/luksa/kubia ...
v3: Pulling from docker.io/luksa/kubia
Digest: sha256:64e06eba2c35e7ccba6f26626ac962164382adedcef30b3416517f47d81f7a0b
Status: Image is up to date for docker.io/luksa/kubia:v3
```
```
DEV [ ~]$ kind load docker-image luksa/kubia:v3
Image: "luksa/kubia:v3" with ID "sha256:e5464484693ec7ccfe5a5309bc98aec3129a4c6d219d38b107e60d6c85d7999a" not present on node "kind-control-plane"
```

```
DEV [ ~]$ kubectl create deploy kubia --image=luksa/kubia:v3
deployment.apps/kubia created
```

```
DEV [ ~]$ kubectl get all
NAME                        READY   STATUS    RESTARTS   AGE
pod/kubia-f578dcdb9-r5sml   1/1     Running   0          14s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   5m13s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/kubia   1/1     1            1           14s

NAME                              DESIRED   CURRENT   READY   AGE
replicaset.apps/kubia-f578dcdb9   1         1         1       14s

```
```
DEV [ ~]$ kubectl describe pod/kubia-f578dcdb9-ngxqx
Name:         kubia-f578dcdb9-ngxqx
Namespace:    default
Priority:     0
Node:         kind-control-plane/
Start Time:  
Labels:       app=kubia
              pod-template-hash=f578dcdb9
Annotations:  <none>
Status:       Running
IP:           
IPs:
  IP:           
Controlled By:  ReplicaSet/kubia-f578dcdb9
Containers:
  kubia:
    Container ID:   containerd://5b76421e24154a668d60c4cdc45e90fab087317bc406311d249c387d8cc792e1
    Image:          luksa/kubia:v3
```
