# Istio

A VirtualService(it does CustomRouting - better name) enables 
us to configure custom routing rules to our service mesh.

k8s does load balancing using service which can have say 3 pods behind it.
Service knows about pods under it with labels and selectors.

Each pods gets a private IP kube-dns will have these to know which pod to direct the call 
to when a client app call a service(kinda load balancing)

So when we apply our YAML for a virtual service,it will go to pilot via Galley.
Galley's responsible for parsing the YAML.

It will then send it to pilot and pilot will distribute 
that across to all of the proxies that need to be changed
as a result of our custom routeing or routing rules.

Therefore, when we go on to make a call
from one pod to another, that traffic is going to be
directed through the envoy proxy,
which will now have that tail end configuration.

And what we're getting here is the benefit of
envoy's traffic management features and envoy is really doing all this canary business.

It's just that Istio gives us the ability to configure all of the proxies that need configuring
without us even really needing to think about the envoy proxies.

### Difference between k8s service and istio service
A Kubernetes service is how we discover the IP addresses of individual pods.

The virtual services allow us to reconfigure the proxies in a dynamic fashion.

And that's why we can do things such as changing the weighting of a canary without having to
restart the system or make any changes to the pods.

```
kind: VirtualService
apiVersion: networking.istio.io/v1alpha3
metadata:
  name: a-set-of-routing-rules-we-can-call-this-anything  # "just" a name for this virtualservice
  namespace: default
spec:
  hosts:
    - fleetman-staff-service.default.svc.cluster.local  # The Service DNS (ie the regular K8S Service) name that we're applying routing rules to.
  http:
    - route:
        - destination:
            host: fleetman-staff-service.default.svc.cluster.local # The Target DNS name
            subset: safe-group  # The name defined in the DestinationRule
          weight: 90
        - destination:
            host: fleetman-staff-service.default.svc.cluster.local # The Target DNS name
            subset: risky-group  # The name defined in the DestinationRule
          weight: 10
```

DestinationRule is a configuration of a load balancer for a particular service.

```
kind: DestinationRule       # Defining which pods should be part of each subset
apiVersion: networking.istio.io/v1alpha3
metadata:
  name: grouping-rules-for-our-photograph-canary-release # This can be anything you like.
  namespace: default
spec:
  host: fleetman-staff-service # Service
  trafficPolicy: ~
  subsets:
    - labels:   # SELECTOR.
        version: safe # find pods with label "safe"
      name: safe-group
    - labels:
        version: risky
      name: risky-group
```
Is it possible to use the weighted destination rules and to make a
single user stick to a canary or the not-canary version depending on which one they got first.

Not out of the box but a work around is using consistent hashing.
https://istio.io/docs/reference/config/networking/destination-rule/#LoadBalancerSettings

consistent hashing - for a given client IP, the hash returned will always be the same which
will make the client to route to the same(canary/non-canary) version.
```yaml
kind: DestinationRule       # Defining which pods should be part of each subset
apiVersion: networking.istio.io/v1alpha3
metadata:
  name: grouping-rules-for-our-photograph-canary-release # This can be anything you like.
  namespace: default
spec:
  host: fleetman-staff-service # Service
  trafficPolicy:
    loadBalancer:
      consistentHash:
      # useSourceIp: true # This won't work, refer below issue link
        httpHeaderName: "x-myval"
  subsets:
    - labels:   # SELECTOR.
        app: staff-service # find pods with label "safe"
      name: all-staff-service-pods

```

https://github.com/istio/istio/issues/9764

So, with weights for canary/non-canary, consistent hashing does not work to ensure sticky sessions 
https://github.com/envoyproxy/envoy/issues/8167

It works if you use httpHeaderName

Consistent hashing can be used to have some instance level cache - sharding, you can use this and achieve cache hits
