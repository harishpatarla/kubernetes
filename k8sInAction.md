# chapter 3

All containers of a pod run on the same node. A pod never spans 2 nodes.
A pod of containers allows you to run closely related processes together and provide them with (almost) the same environment as if they were all running in a single container while keeping them somewhat isolated. This way, you get the best of both worlds. You can take advantage of all the features containers provide, while at the same time giving the processes the illusion of running together.

THE PARTIAL ISOLATION BETWEEN CONTAINERS OF THE SAME POD - You want containers inside each group to share certain resources, although not all, so that they’re not fully isolated. Kubernetes achieves this by configuring Docker to have all containers of a pod share the same set of Linux namespaces instead of each container having its own set.

UNDERSTANDING HOW CONTAINERS SHARE THE SAME IP AND PORT SPACE - Containers of different pods can never run into port conflicts because each pod has a separate port space. All the containers in a pod also have the same loopback network interface, so a container can communicate with other containers in the same pod through localhost.
pods are logical hosts and behave much like physical hosts or VMs in the non-container world. Processes running in the same pod are like processes running on the same physical or virtual machine, except that each process is encapsulated in a container.

You should think of pods as separate machines, but where each one hosts only a certain app. Because pods are relatively lightweight, you can have as many as you need without incurring almost any overhead.

SPLITTING MULTI-TIER APPS INTO MULTIPLE PODS(front end and backend database) 
If you have 2 nodes cluster, spitting the pod into 2 makes sense as you can make use of both the nodes and its infrastructure like CPU and memory.  
Scaling is another reason you shouldn't put them both on single pod. k8s scales whole pods. FE has different scaling requirements than backend.
Pods should contain tightly coupled containers, usually a main container and containers that support the main one.

DECIDING WHEN TO USE MULTIPLE CONTAINERS IN A POD
Do they need to be run together or can they run on different hosts?
Do they represent a single whole or are they independent components? 

 Must they be scaled together or individually?

THE MAIN PARTS OF A POD DEFINITION
Metadata includes the name, namespace, labels, and other information about the pod.

Spec contains the actual description of the pod’s contents, such as the pod’s containers, volumes, and other data.

Status contains the current information about the running pod, such as what condition the pod is in, the description and status of each container, and the pod’s internal IP and other basic info.


```
kubectl explain pods	
kubectl explain pod.spec	
kubectl create -f kubia-manual.yaml	
kubectl get po kubia-manual -o yaml	
kubectl get po kubia-manual -o json	
kubectl get pods	
docker logs <container id>	
kubectl logs kubia-manual	
kubectl logs kubia-manual -c kubia	
```
if additional containers exist in the pod, you’d have to get its logs like this.
````
kubectl port-forward kubia-manual 8888:8080	
kubectl create -f kubia-manual-with-labels.yaml	
kubectl get po --show-labels	
kubectl get po -L creation_method,env	
kubectl label po kubia-manual creation_method=manual	Labels can also be added to and modified on existing pods
kubectl label po kubia-manual-v2 env=debug --overwrite	need to use the --overwrite option when changing existing labels.
kubectl get po -l creation_method=manual	To see all pods you created manually
kubectl get po -l env	To list all pods that include the env label
kubectl get po -l '!env'	those that don’t have the env label
kubectl label node gke-kubia-85f6-node-0rrx gpu=true	add label to a node needing GPU computing
kubectl get nodes -l gpu=true	
kubectl get po kubia-zxzij -o yaml	
kubectl annotate pod kubia-manual mycompany.com/someannotation="foo bar"	adding an annotation to your kubia-manual pod
kubectl describe pod kubia-manual	
kubectl get ns	list all namespaces in your cluster
kubectl get po --namespace kube-system	to list pods in that namespace only. You can also use -n instead of --namespace.
kubectl delete po kubia-gpu	
kubectl delete po -l creation_method=manual	
delete pods by using a label selector

kubectl delete po -l rel=canary	
delete all canary pods at once by specifying the rel=canary label selector

kubectl delete ns custom-namespace	delete the whole namespace(the pods will be deleted along with the namespace automatically)
kubectl delete po --all	
delete all pods in the current namespace

kubectl delete all --all	The first all in the command specifies that you’re deleting resources of all types, and the --all option specifies that you’re deleting all resource instances instead of specifying them by name.
````

A label is an arbitrary key-value pair you attach to a resource, which is then utilized when selecting resources using label selectors. A resource can have more than one label, as long as the keys of those labels are unique within that resource.

Each pod is labeled with two labels:

app, which specifies which app, component, or microservice the pod belongs to.

rel, which shows whether the application running in the pod is a stable, beta, or a canary release.
A label selector can select resources based on whether the resource
 Contains (or doesn’t contain) a label with a certain key
 Contains a label with a certain key and value
 Contains a label with a certain key, but with a value not equal to the one you specify.

A selector can also include multiple comma-separated criteria. Resources need to match all of them to match the selector.

Using labels and selectors to constrain pod scheduling
chapter 4

kubelet checks for the app status and brings it up if down, so you have high availability without you doing anything.
java OutOfMemoryError has the JVM still running but the app is not healthy, in such cases it can't know the app is down.
We cannot add checks for these errors in the application and shut down the app, then kubelet will notice and bring it back up because this cannot be done for cases like infinite loop and deadlock. we need to check its health externally
specifying a liveness probe for each container, k8s will execute the probe and restart container id probe fails.
HTTP GET probe(other than 2xx,3xx, probe fails, restart container)
TCP Socket probe
An Exec probe
```
Kubectl get po kubia-liveness(command to check liveness probe)
Kubectl logs mypod --previous(previous container logs)
Kubectl describe po kubia-liveness(to describe the pod)
```
When a container is killed, a completely new container is created—it’s not the same container being restarted again.
you can add additional properties to the liveness probe.
delay - After how long probing starts after the container is up.
timeout - the container must return a response in (timeout) seconds or the probe is counted as failed
period - app is probed every (period) seconds
initialDelaySeconds - to set an initial delay to account for your app’s startup time.

Creating effective liveness probes:

(/health, have the app perform an internal status check of all the vital components running inside the app to ensure none of them has died or is unresponsive.

KEEPING PROBES LIGHT

DON’T BOTHER IMPLEMENTING RETRY LOOPS IN YOUR PROBES

ReplicationControllers

A ReplicationController’s job is to make sure that an exact number of pods always matches its label selector.
 A label selector, which determines what pods are in the ReplicationController’s scope
 A replica count, which specifies the desired number of pods that should be running
 A pod template, which is used when creating new pod replicas.

A pod instance is never relocated to another node. Instead, the ReplicationController creates a completely new pod instance that has no relation to the instance it’s replacing.
```
kubectl get po -l creation_method=manual
```
The first all in the command specifies that you’re deleting resources of all types, andthe --all option specifies that you’re deleting all resource instances instead of specifyingthem by name
