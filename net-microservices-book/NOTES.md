# Intro to Containers and Docker
```Container image``` is a package with all the dependencies and information needed to create a container. An image includes all the dependencies (such as frameworks) plus deployment and execution configuration to be used by a container runtime. Usually, an image derives from multiple base images that are layers stacked on top of each other to form the container's filesystem. An image is immutable once it has been created.  
```Dockerfile``` is a text that contains instructions how to build Docker image.  
```Container``` is an instance of a Docker image.  
```Volumes``` offer a writable filesystem. Since images are read-only but most programs need to write to the filesystem, volumes add a writable layer, on top of the container image, so the programs have access to a writable filesystem.  
```Multi-stage build``` is a feature that helps to reduce the size of the final images. For example, a large image containing SDK can be used to compile and publish and than a small runtime-only base image can be used to host the app.  


* You can debug inside docker container your app. VS Studio defenitely allows it.
* Sometimes your infrastructure will affect dockerfile. But in case you are working with different technologies stacks, e.g python + .net, it's nice to have dockerfiles to be independet from your infrastructure. Than you can treat .net and python apps in docker containers as black box and you will not need to doo any additional work for configuration.

docker run --rm (it will remove the container after it is killed, otherwise your system will grow with trash containers)

# Architecting container and microservice-based applications
In most cases you can think of a container as an instance of a process. A process does not maintain persistent state. Containers are not durable. You should always assume that container images, the same way like processes, have multiple instances or will eventually be killed. So the following solutions are used to manage data in Docker applications:
* ```Volumes``` are stored in an area of the host filesystem thats managed by Docker.

Or by using remote storages:
* Remote relational database like ```MS SQL``` or ```NoSQL``` database like ```CosmosDB```, or cache service like ```Redis```.

However, always have into account that container design by default is stateless, and designing a system which will use local storage ```Docker Volumes``` is conflicting. Tip: use remote storages if possible.  

Q: Size of microservice? Small enough but not too small so it will create dependencies on other microservices, therefore making it a ```big ball of mad``` (removing independent deployment feature). Low coupling and high cohesion.  

## Data sovereignty per microservice
An important rule for microservices architecture is that each microservice must own its domain data and logic. This principle is similar in ```DDD``` where each ```Bounded Context``` must own its domain model (behaviour plus logic and data). Each DDD Bounded Context correlates to one business microservice(one or several one).  
In monolith applications (especially with layared architecture) you usually have one (or few) databases. The centralized database approach initally looks simpler and seems to enable reuse of entities in different subsystems to make everything consisntent. But the reality is ```you end up with huge tables that serve many different subsystems, and that include attributes and columns that aren't needed in most cases.```  
A monolithical applications with typically a single relational database has two important benefits:
* ACID transacitons
* SQL language

! Data access becomes much more complicated when you move to a microservices architecture. The ```data in the microservice should be encapsulated. Private and not accessible for other except from public APIs(REST,gRPC, AMQP, etc).```  

Q: On your own words what is Bounded Context?
A: In those younger days we were advised to build a unified model of the entire business. But DDD show us that 'total unification of the domain model for a large system will not be cost effective and sucks'. So instead DDD divides up a large system into Bounded Contexts, each of which can have a unified model (usually connected with other subdomain, bounded context by entity id).  

Q: How to create queries that retrieve data from several microservices?
A: First of all, it always depends on the complexety of query. But in any case, you will need to aggregate the information if you want to improve the efficiency in the communication of your system. The most popular solutions are the following:
* ```API Gateway```. Aggregate by using API Gateway. However, you need to be careful about implementing this pattern, because it can be a choke point in your system, and it can violate the principle of the microservice autonomy.
* ```CQRS with query tables```. Another solution is the ```materialized view pattern```. In this approach, you generate, in advance (prepare denormalized data before query even happen), a read-only table with the data that's owned by multiple microservices. This approach not only solves the original problem (query across microservicds), but it can also improves performance considerably when comparing with a compex join. Of course, using CQRS with query tables means additional development work, and you'll need to emberace ```eventual consistency```.


***CAP Theorem*** states that ```any distributed system can only provide two of the following three```:
* ```Consistency```: every read receives the most recent write or error.
* ```Availability```: every request should receive response (not error).
* ```Partition tolerance```: the system continues to operate despite some number of messages being dropped (or delayed) by the network between nodes.

So because we already use distributed system (and usually want to have partion tolerance) we need to choose between availability and ACID strong consistency.  
A good solution for the consistency problem is eventual consistency,  event-driven communication plus publish-and-subscribe system.  

Q: How did you improve performance in microservices?  
A: First example, moved our HTTP chain of calls (which is bad practice, but was initally decided to go this way for simplicity) to the message brokers. Clinet => (http) => UserService => (http) => UserIdentityService => (http) => NotificaitonService etc.  

Q: How to design communication between microservices?
A: Chain of http is big no (blocking and low performance, coupling microservices with http, failure in any of the microserivces). Actually two possible solutions:
* Async messages using pub/sub
* async HTTP (with retries exponential backoffs and circuit breaker mechanism) polling (using some chron job to raise an call) independently from the original HTTP request/response cycle.

# API Gateway
Usually multiple and different client apps will call your API Gateway. That fact can be an important risk because your API Gateway service will be growing and evolving based on many different requirements from the client app. Eventually, it will be bloated bacause of those different needs and it can become similar to a monolithic application (monolithic service). That's why ```it is very much recommended to split the API Gateway in multiple services or multiple smaller API Gateways, one per client app for instance```.