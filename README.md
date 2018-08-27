# Service-Mesh

   In micro-services era, a typical application might encompass hundreds of services; each service might run in multiple instances; and each of those instances might be in a constantly-change state when they are deployed / scheduled by using an orchestrator.  Currently, in an organization, a routine set successfully in making systematize pattern for packaging and deploying the services with the powerful abstractions provided by things like Docker and Kubernetes which drastically reduced the incremental operational burden to deployment. With these tools, it results in a dramatic reduction in the cost of adopting micro-services.


The real challenge with the micro-services type implementation is not building the services themselves, but the communication between services and different confronts arising from this inter-service communications are

*	Load balancing
*	Distributed tracing
*	TLS encryption and mutual TLS authentication
*	Resiliency
    *	Retry
    *	Circuit Breaker
*	Service Versioning
even more…

Is it possible to handle all these challenges in a decentralized approach without touching the application services? 

Does it practicable by offloading all of the inter-service communication requirements to a different layer by keeping service code independent as these specifications are quite generic in any micro-services implementations?

Can we undertake and provide language agnostic solution as an application service may code in different languages and communication among those might on the top of standard protocols such as HTTP1.x/2.x,gRPC, WebSocket, TCP traffic etc..?

Does it feasible to face these challenges on already deployed / on the top of micro-services network without touching it?

### To address all these concerns, a “Service Mesh” is designed 

* A service mesh is a dedicated infrastructure layer for handling service-to-service communication and global cross-cutting of concerns to make these communications more reliable, secure, observable and manageable

* This design helped in standardizes the runtime operations of our applications in the same way that Docker and Kubernetes standardized deploy-time operations

* A given micro-service won’t directly communicate with the other micro-service. Rather all service-to-service communications will take place on-top of a software component called side-car proxy.

### Common features offered from Service Mesh

![image](https://user-images.githubusercontent.com/20100300/44647208-06c10a80-a9fb-11e8-8391-e51444887529.png)

 The different functionalities offered through these features are
 
 * It enhance reliability by supporting circuit-breaking, retries and timeouts, fault injection, fault handling, load balancing and failover
 
 * It not only provides discovery of service through a dedicated service registry but also tracking the messages that convey information about server locations. For instance, it guarantees message passing  and proper return of errors when message don’t  reach their destinations
 
 * Primitive routing capabilities
 
 * It improves deep visibility by providing long-term metrics on service availability. Also it provides capabilities like monitoring, distributed logging, distributed tracing
 
 * Transport Level Security and key management
 
 * Simple blacklist and whitelist based access control
 
 * Native support for containers, Docker and Kubernetes
 
 * It also often has more complex operational requirements, like A/B testing, canary releases
 
 In service mesh, for a given service which communicates with other service comprises of 
 
 * Business Logic
    * Includes logic related to its business functions, computations
 * Primitive Network Functions
    * A basic high-level network interactions to connect with the service mesh / side-car proxy
 * Application Network Functions
    * Tightly coupled to network such as circuit breaking, timeouts, retries, client side LB etc
 * Control Plane
    * Quite useful to support capabilities like access control, observability, service discovery etc
    
![untitled](https://user-images.githubusercontent.com/20100300/44647451-bac29580-a9fb-11e8-8a4f-3d600494d618.png)
   
### Service mesh proxy deployment models

It can be deployed in two different patterns

* Per-host proxy deployment
    *	One proxy is deployed per host
    *	A host can be virtual machine, or a physical host or a Kubernetes worker node
    *	Multiple instances of application services run on the host
    *	On a given host route traffic through this one proxy instance.
    *	In case of kubernetes, the proxy instance can be deployed as daemonset
   
* Sidecar proxy deployment
    * One proxy is deployed per instance of every service
    * In kubernetes, a service mesh sidecar container can be deployed along with application service container as a part of the kubernetes pod
    * This approach requires more instances of sidecar, hence a smaller resource profile for sidecar is usually appropriate
    
### Request Flow – Service mesh, ingress, egress

![image](https://user-images.githubusercontent.com/20100300/44647562-fbbaaa00-a9fb-11e8-94d4-a1edb3b140dd.png)

*	By default, proxies handle only intra-service mesh cluster traffic – between the source (upstream) and the destination (downstream) services. 
*	To expose a service which is part of service mesh to outside world, it must enable ingress traffic
*	Similarly, if a service depends on an external service , it requires enabling the egress traffic

### Service Mesh implementations

Linkerd and Istio are two popular open source service mesh implementations. As both follow a similar architecture, but with different implementation mechanism, will explore more with Istio and will give insight about the features provided by these two implementations

### Istio

Without any changes in the service code, it eases creation of network of deployed services with

*	Fine-grained control of traffic behavior with rich routing rules, retries, failovers, and fault injection
*	Automatic load balancing for HTTP, gRPC, WebSocket, and TCP traffic
*	A pluggable policy layer and configuration API supporting access controls, rate limits and quotas
*	Automatic metrics, logs and traces for all traffic with in a cluster, including ingress and egress
*	Secure service-to-service communication in a cluster with strong identity-based authentication and authorization

### Architecture

Logically split into a 

* data plane
    * Touches every packet/request in the system
    * Responsible for 
        * service discovery 
        *	health checking 
        *	routing
        *	load balancing
        *	authentication / authorization
        *	observability
        
*	control plane
    *	provides policy and configuration for all the running data planes in the mesh
    *	does not touch any packets / requests in the system 
    *	it turns all of the data planes into a distributed system

Different features provided by Istio and Linkerd explained(in detail) in the below table

Features  | Istio  |  Linkerd
----------| -------| ----------
Proxy | Envoy | Finagle + Jetty
Circuit Breaker | Enforces at network level | Two types :-Fail Fast (session-driven) / Failure Accrual (request-driven)
Dynamic Request Routing | To service instances by versions or environment | Service destination (service name) and the concrete destination (version and environment) 
Traffic Shifting/Splitting | Yes | Yes
Service Discovery | Platform-agnostic service discovery | Namer – file based service discovery;can also work with Zookeeper, Consul, Kubernetes
Load Balancing | Envoy’s load balancing algorithms. Eject unhealthy service instance from the load balancing pool | Finagle’s load balancing algorithms.Provides failure-and latency-aware load balancing
Security | Securing - End-user to service / Service-to-Service | Easy to add TLS to all service-to-service calls; Per-service/Per-environment certificates; Key management process
Access Control | Enforce simple whitelist – or blacklist-based access control; Enforce quotas and rate limits | -
Observability | Metrics(Prometheus, statsd); Monitoring(New Relic, Stackdrive); Logging(Application and access logs); Distributed tracing(zipkin) | Distributed tracing (Zipkin); Metrics(InfluxDB, Prometheus, statsd)
Deployment support | Kubernetes; Sidecar proxy | Kubernetes, Mesos, Cluster of hosts; per-host or sidecar proxy
Control Plane | Pilot, Mixer, Istio-Auth | Namerd
Service-to-Service communication | HTTP1.1 or HTTP/2, gRPC or TCP with or without TLS | HTTP1.1 or HTTP/2, gRPC or TCP with or without TLS



