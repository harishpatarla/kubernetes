```
# --save-config to Store current properties in resource's annotation,--record to record the command in the deployment revision history
kubectl create -f file.deployment.yml --save-config --record
kubectl apply -f file.deploymeent.yml 
kubectl scale deployment [deployment-name] --replicas=5
kubectl scale -f file.deployment.yml --replicas=5
```

k8s strengths - zero-downtime deployments out of the box

update applications pods without impacting end users
- **Rolling updates**
- **Rollbacks** 
- **Canary deployments**
- **Blue green deployments**

**Rolling updates:**
- RS increase new pods while decreasing old pods
- Service handles load balancing traffic to available pods
- Rolling update is default strategy, Recreate - downtime

```yaml
spec:
  replicas: 4
  minReadySeconds: 1 # Default 0, how long to wait to be considered healthy(gives startup seconds), you can also configure probes
  progressDeadlineSeconds: 60 # Default 600s(seconds to wait before reporting stalled deployment)
  revisionHistoryLimit: 5  #Default 10(No. of RS that can be rolled back)
  strategy:
    type: RollingUpdate # This is the default. other type is recreate-will have downtime
    rollingUpdate:
      maxSurge: 1  #Default 25%, max pods that can exceed the replicas count.
      maxUnavailable: 1  # Default 25%, max pods that are not operational.
```

**Rollbacks:** 
```yaml
# Check status of the deployment
kubectl rollout status deployment [deployment-name]

# Rollback a deployment 
kubectl rollout undo -f file.deployment.yml

# Rollout history deployments
kubectl rollout history deployment [deployment-name]

# Rollback to a specific revision
kubectl rollout undo deployment [deployment-name] --to-revision=2
```

**Canary deployments:**

Strategy to check the viability of deployment before releasing it to wider audience

```yaml
# Create service, stable deployment, canary deployment

```



