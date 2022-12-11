# O𝐫𝐜𝐡𝐞𝐬𝐭𝐫𝐚𝐭𝐢𝐨𝐧 and C𝐡𝐨𝐫𝐞𝐨𝐠𝐫𝐚𝐩𝐡𝐲

![image](https://user-images.githubusercontent.com/115500959/206921954-3c44c5e0-8e55-458d-a2ca-6841643bb555.png)


## Details Of 𝐨𝐫𝐜𝐡𝐞𝐬𝐭𝐫𝐚𝐭𝐢𝐨𝐧 and 𝐜𝐡𝐨𝐫𝐞𝐨𝐠𝐫𝐚𝐩𝐡𝐲

How do microservices collaborate and interact with each other?

There are two ways: 𝐨𝐫𝐜𝐡𝐞𝐬𝐭𝐫𝐚𝐭𝐢𝐨𝐧 and 𝐜𝐡𝐨𝐫𝐞𝐨𝐠𝐫𝐚𝐩𝐡𝐲.

The diagram below illustrates the collaboration of microservices.

Choreography is like having a choreographer set all the rules. Then the dancers on stage (the microservices) interact according to them. Service choreography describes this exchange of messages and the rules by which the microservices interact.

Orchestration is different. The orchestrator acts as a center of authority. It is responsible for invoking and combining the services. It describes the interactions between all the participating services. It is just like a conductor leading the musicians in a musical symphony. The orchestration pattern also includes the transaction management among different services.

**The benefits of orchestration:** <br>
**1. Reliability -** orchestration has built-in transaction management and error handling, while choreography is point-to-point communications and the fault tolerance scenarios are much more complicated. <br>

**2. Scalability -** when adding a new service into orchestration, only the orchestrator needs to modify the interaction rules, while in choreography all the interacting services need to be modified.<br>

**Some limitations of orchestration:** <br>
**1. Performance -** all the services talk via a centralized orchestrator, so latency is higher than it is with choreography. Also, the throughput is bound to the capacity of the orchestrator. <br>

**2. Single point of failure -** if the orchestrator goes down, no services can talk to each other. To mitigate this, the orchestrator must be highly available.<br>

**Real-world use case:** Netflix Conductor is a microservice orchestrator and you can read more details on the orchestrator design.


![image](https://user-images.githubusercontent.com/115500959/206841373-ffcb362e-a488-4af1-9e2e-b33bb1569260.png)

## **SAGA Pattern -** 

SAGA pattern is nothing but Co-ordination techniqueue(Orchestration Techniqueue or Coreography Techniqueue).

### Orchestration Techniqueue

This Technique we can solve via two channels 
1. Command Channel
2. Reply Channel

![image](https://user-images.githubusercontent.com/115500959/206921862-3f1a4fb7-d430-4f01-8264-532c5a5992aa.png)

### Choreography Techniqueue

![image](https://user-images.githubusercontent.com/115500959/206921786-228d3ab9-b726-45ef-9b9d-e409a3d5d812.png)

