**SOA**     
Inter-service communication - Smart pipes, such as Enterprise Service Bus, using heavyweight protocols, such as SOAP and the other WS* standards.
Data - Global data model and shared databases
Typical service - Larger monolithic application

**Microservices**
Inter-service communication - Dumb pipes, such as a message broker, or direct service-to-service communication, using lightweight protocols such as REST or gRPC
Data - Data model and database per service
Typical service - Smaller service


* It enables the continuous delivery and deployment of large, complex applications.
* Services are small and easily maintained.
* Services are independently deployable.
* Services are independently scalable.
* The microservice architecture enables teams to be autonomous.
* It allows easy experimenting and adoption of new technologies.
* It has better fault isolation.

It has the testability required by continuous delivery/deployment

 Drawbacks of the microservice architecture


The patterns are also divided into three layers:

Infrastructure patterns— These solve problems that are mostly infrastructure issues outside of development.
Application infrastructure— These are for infrastructure issues that also impact development.
Application patterns— These solve problems faced by developers.


Communication patterns
* Communication style— What kind of IPC mechanism should you use?
* Discovery— How does a client of a service determine the IP address of a service instance so that, for example, it makes an HTTP request?
* Reliability— How can you ensure that communication between services is reliable even though services can be unavailable?
* Transactional messaging— How should you integrate the sending of messages and publishing of events with database transactions that update business data?
* External API— How do clients of your application communicate with the services?
