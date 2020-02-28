# k8s with kind

## If you are not using minikube but using kind then these steps can help with running your k8s workloads:
After installing Kind and creating the kind cluster we might want to quickly start spinning workloads in k8s cluster. 
For that we need docker images if we are not building one.

For that we need to do the below steps which comes from [kind](https://kind.sigs.k8s.io/docs/user/quick-start/) 
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

Below is a sample image kubia from k8s in action book:
```
docker pull luksa/kubia:v4
kind load docker-image luksa/kubia:v4
kubectl create deploy kubia --image=luksa/kubia:v4
```

