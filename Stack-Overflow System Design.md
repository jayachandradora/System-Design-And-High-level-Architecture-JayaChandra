# Stack-Overflow-Architecture
Stack Overflow Architecture - High Level Design
![image](https://user-images.githubusercontent.com/115500959/195127693-2d514fbd-2a6b-48d8-b45e-7ea74cf202de.png)


## Details of Explanations

๐๐ก๐๐ญ ๐ฉ๐๐จ๐ฉ๐ฅ๐ ๐ญ๐ก๐ข๐ง๐ค ๐ข๐ญ ๐ฌ๐ก๐จ๐ฎ๐ฅ๐ ๐ฅ๐จ๐จ๐ค ๐ฅ๐ข๐ค๐ <br>
The interviewer is probably expecting something on the left side.<br>
1. Microservice is used to decompose the system into small components.<br>
2. Each service has its own database. Use cache heavily.<br>
3. The service is sharded.<br>
4. The services talk to each other asynchronously through message queues.<br>
5. The service is implemented using Event Sourcing with CQRS.<br>
6. Showing off knowledge in distributed systems such as eventual consistency, CAP theorem, etc.<br>

๐๐ก๐๐ญ ๐ข๐ญ ๐๐๐ญ๐ฎ๐๐ฅ๐ฅ๐ฒ ๐ข๐ฌ <br>
Stack Overflow serves all the traffic with only 9 on-premise web servers, and itโs on monolith! It has its own servers and does not run on the cloud.<br>
