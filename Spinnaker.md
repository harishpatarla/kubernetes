**Spinnaker terminology**

Account - contains an account name and credential.
 
    Credentials to docker registeries where docker images are stored
    Credentials to access to kubernetes-api server

_Instance_ - pod

_Server group_ - group of instances

_Cluster_ - A spinnaker deployment, logical grouping of server groups.

**Providers** 

    Set of virtual resources spinnaker has control over. 
    
    Provider is where you deploy your resources(server groups)
    
    Server group is what spinnaker calls source of deployable artifacts
    
    You deploy applications on server groups which are tied to cloud provider

**Deployment Pipeline**
    
    Can be triggered automatically by Jenkins

**Immutable Infrastructure** 

    Consists of immutable components that are replaced on every deployment rather than being updated in-place(pets vs cattle)
    Immutable components are easier to validate and test as we use same image across infrastructure.
    Images are baked by packer software 


**Building pipelines**
    
    Pipeline executes one or more stages with specific stage type.
    Every stage, except for first, has a dependency on previous stage.
    You can run stages in parallel by defining same dependency i.e. previous stage.
    You can declare more than one dependency per stage(which have to complete before the stage is started).
    You edit in UI but there is a generated json which can also be edited

**Stage types**

![alt text](https://github.com/harishpatarla/kubernetes/blob/master/images/pipelines.png)

![alt text](https://github.com/harishpatarla/kubernetes/blob/master/images/pipelines2.png)

![alt text](https://github.com/harishpatarla/kubernetes/blob/master/images/pipelines3.png)

![alt text](https://github.com/harishpatarla/kubernetes/blob/master/images/pipelines4.png)

**Manual Decisions**
