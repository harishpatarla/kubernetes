
####  Ch 4: Managing Communication with your microservices
 Ingress vs Service vs Service Mesh
 
 * Management of internal and/or external cluster communications
 * Ingress and service kubernetes objects
 * Ingress Controllers - traffic from internet to service
 
**Ingress and Service**

* Can provide load balancing, SSL termination and name-based virtual hosting.
* A service can be exposed internally or externally without an ingress(kubectl expose).

_**Kubectl expose**_ - can expose a deployment, repication controller, replica set or pod as service object
* ClusterIP, NodePort, LoadBalancer, or ExternalName
* ClusterIP - default (internal service only accessible by other objects in k8s)
* can expose a resource externally

 _**Ingress Controller**_ 
* Required If the Ingress object is to function properly
* you can use more than one ingress controller on a cluster
* Common controllers include 
    1.  Azure Application Gateway(Platform managed)
    2.  NGINX(Deployment managed)
    3.  Istio and Traefik(Framework managed)
 
 


