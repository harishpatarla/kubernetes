Istio Architecture

![alt text](https://github.com/harishpatarla/kubernetes/blob/master/images/istioarchitecture.png)

below is usual comms between services will vanilla k8s.
![alt text](https://github.com/harishpatarla/kubernetes/blob/master/images/k8scomms.png)

In above image, 
1. service - links host to IP
2. pod - Add or remove pods
3. container - container serves requests

![alt text](https://github.com/harishpatarla/kubernetes/blob/master/images/virtualservice.png)

**VirtualService** -  We can apply custom routing rules where we can say any requests coming from specific hosts can be redirected to  
specific destinations. It's an istio service which replaces k8s Service.

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

**_we can configure retries, timeouts, etc.
Istio is control plane for communication between services.
    
Istio can be default as containers in k8s cluster which can be injected as proxies.
You can configure your namespace to automatically inject istio proxies to any containers._**


#### Managing service Traffic - Blue/Green,  Canary deployments, Dark launch

**Dark launch** - only test team has access to it although its deployed in production.
After QA is happy we can either do Blue/Green or Canary.

**Blue/Green** - We have 2 versions of app running in a cluster. We can switch the app to new version. Its an all or nothing approach.

**Canary** - we can shift traffic based on percentage. most traffic sent to v1 until we gain confidence in v2 and then do staged rollout. 
 
**Circuit Breaker** - if one instance fails then traffic is stopped giving the instance time to recover.

**DestinationRule** - It works along with virtualservice where we can have subsets of traffic directed based on labels and selectors. 

![alt text](https://github.com/harishpatarla/kubernetes/blob/master/images/DestinationRule.png)

**Dark launch Example** -
We have VirtualService and DestinationRule defined below.

Any pods with label v1 will go to v1 subset and v2 will go to v2 subset.

Virtual service - For hostname reviews, any requests for reviews service will go to virtual service instead of k8s service.
And below rules will apply where header matches tester, v2 version is shown. For all other requests v1.

In real world you would use cookies or JWT to do this.

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  trafficPolicy:
    tls:
      mode: ISTIO_MUTUAL
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - match:
    - headers:
        end-user:
          exact: tester
    route:
    - destination:
        host: reviews
        subset: v2
  - route:
    - destination:
        host: reviews
        subset: v1
``` 
**Delay Injection example**

For tester, we can have fixed delays introduced too which can help us test app is fault tolerant and behaves as expected.  

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - match:
    - headers:
        end-user:
          exact: tester    
    fault:
      delay:
        percent: 100
        fixedDelay: 2.5s
    route:
    - destination:
        host: reviews
        subset: v2
  - route:
    - destination:
        host: reviews
        subset: v1
```

**Fault Injection Example**
Below fault condition will give 50 % calls a 503 response. 
We can test if our retry functionality behaves as expected. 
```yaml
spec:
  hosts:
  - reviews
  http:
  - match:
    - headers:
        end-user:
          exact: tester    
    fault:
      abort:
        percent: 50
        httpStatus: 503
    route:
    - destination:
        host: reviews
        subset: v2
  - route:
    - destination:
        host: reviews
        subset: v1

```
