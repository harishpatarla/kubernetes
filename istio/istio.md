Istio Architecture

![alt text](https://github.com/harishpatarla/kubernetes/blob/master/images/istioarchitecture.png)

below is usual comms between services will vanilla k8s.
![alt text](https://github.com/harishpatarla/kubernetes/blob/master/images/k8scomms.png)

In above image, 
1. service - links host to IP
2. pod - Add or remove pods
3. container - container serves requests

![alt text](https://github.com/harishpatarla/kubernetes/blob/master/images/virtualservice.png)

_**VirtualService**_ -  we can apply custom routing rules where we can say any requests coming from specific hosts can be redirected to  
specific destinations. 

Also, we can define the percentage of requests that can go to different versions based on labels.

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: details
spec:
  hosts:
  - details
  http:
  - route:
    - destination:
        host: details
```

Fault tolerance, Client timeouts and automatic retries

Fault tolerance where first call to details fails after waitign for 30 seconds

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: details-v1
  labels:
    app: details
    version: v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: details
      version: v1
  template:
    metadata:
      labels:
        app: details
        version: v1
    spec:
      serviceAccountName: bookinfo-details
      containers:
        - name: details
          image: docker.io/sixeyed/bookinfo-details:v1
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 9080
          env:
            - name: SERVICE_VERSION
              value: v-timeout-first-call

```

Add timeouts - 5 seconds.
 
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: details
spec:
  hosts:
  - details
  http:
  - route:
    - destination:
        host: details
    timeout: 5s
```

Retry if calls fail

I would do 2 reties if some 5xx happens. so retry won't happen for 404 errors etc.
Waits for 5 sec per try and here we can observe that the request Id remains the same.
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: details
spec:
  hosts:
  - details
  http:
  - route:
    - destination:
        host: details
    retries:
      attempts: 2
      perTryTimeout: 5s
      retryOn: 5xx
```
