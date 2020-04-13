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

**Custom Routing Rules**

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

Istio is managing external traffic in this case as there is a front end app calling this service.

So we need an extra component called Gateway which accepts all external traffic and routes them to virtualservice.

**Blue/Green example**

![alt text](https://github.com/harishpatarla/kubernetes/blob/master/images/bluegreen.png)

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
    - bookinfo.local
  gateways:
    - bookinfo-gateway
  http:
  - route:
    - destination:
        host: productpage
        subset: v1
        port:
          number: 9080
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo-test
spec:
  hosts:
    - test.bookinfo.local
  gateways:
    - bookinfo-gateway
  http:
  - route:
    - destination:
        host: productpage
        subset: v2
        port:
          number: 9080
```

In real world you would have labels - _blue/green_ and can use them to switch. 

So we never change anything in k8s workload yamls. This happen in layer 7.


**Canary example**

![alt text](https://github.com/harishpatarla/kubernetes/blob/master/images/canary.png)

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
    - bookinfo.local
  gateways:
    - bookinfo-gateway
  http:
  - route:
    - destination:
        host: productpage
        subset: v1
        port:
          number: 9080
      weight: 70
    - destination:
        host: productpage
        subset: v2
        port:
          number: 9080
      weight: 30
```
**Canary with sticky sessions**
Since a user can see randomly v1/v2 when he requests, we can leverage cookies.

So a user who sees v2, will always see v2 with the below conf.

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
    - bookinfo.local
  gateways:
    - bookinfo-gateway
  http:
    - match:
        - headers:
            cookie:
              regex: "^(.*?;)?(product-page=v2)(;.*)?$"
      route:
        - destination:
            host: productpage
            subset: v2
            port:
              number: 9080
    - route:
        - destination:
            host: productpage
            subset: v1
            port:
              number: 9080
          weight: 70
        - destination:
            host: productpage
            subset: v2
            port:
              number: 9080
          weight: 30
```

**Circuit Breaker with Outlier Detection**

one of the pod evicted from the group and requests stop being sent to it, giving it time to recover. 

![alt text](https://github.com/harishpatarla/kubernetes/blob/master/images/canary.png)

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: details
spec:
  host: details
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
      trafficPolicy:
        outlierDetection:
          consecutiveErrors: 2 #how many errors in 1m interval
          interval: 1m
          baseEjectionTime: 5m #keep them evicted for this time. 1st eviction for 5m, 2nd for 10m, 3rd for 15m
          maxEjectionPercent: 100 #this would evict all the pods
```

```
kubectl describe dr <resourcename>  
kubectl describe vs <resourcename>  
kubectl describe svc <resourcename>  

```

#### Securing communication with Mutual TLS

_**Istio**_ - 

    can change the nature of communication between services, 
    upgrade basic http to https with client certificates,
    without needing to change any of the code or managing certs ourselves.

_**Trusted subsystems pattern**_ - When services are internal and not publicly available its tempting not to encrypt communication between those services.
So there is no authentication, authorization or encryption of traffic.
Certificates need not be part of the code. We can make use of the certificates for authentication and authorization which istio manages for us.

Each client and each service has a certificate that is managed by istio .
We can configure rules to apply policies where we can service1 can be accessed by service2 but not service3.

_**JWT**_ - This same thing can be used for end user authentication where we require end user
to provide a token to access our service and use that token foe authentication/authorization.

![alt text](https://github.com/harishpatarla/kubernetes/blob/master/images/mTLS.png)

_**Mesh policy**_ - entire service mesh (default) - across all services in all namespaces

**_Policy_** - More on specific service level

peers - specifies authentication mechanism for calling service

TLS - server has certificate to identify itseld and encrpty traffic but also client also has 
certificate to identify itself

mode - PERMISSIVE - if both sides of communication is managed by istio.
This means mTLS is supported but is not required.

DestinationRule - tls trafficMode - ISTIO_MUTUAL - istio will manage the traffic
Istio proxy upgrade traffic from http to https for a calling service.


