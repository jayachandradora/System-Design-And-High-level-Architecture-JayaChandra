# Stack-Overflow-Architecture
Stack Overflow Architecture - High Level Design
![image](https://user-images.githubusercontent.com/115500959/195127693-2d514fbd-2a6b-48d8-b45e-7ea74cf202de.png)


## Details of Explanations

𝐖𝐡𝐚𝐭 𝐩𝐞𝐨𝐩𝐥𝐞 𝐭𝐡𝐢𝐧𝐤 𝐢𝐭 𝐬𝐡𝐨𝐮𝐥𝐝 𝐥𝐨𝐨𝐤 𝐥𝐢𝐤𝐞 <br>
The interviewer is probably expecting something on the left side.<br>
1. Microservice is used to decompose the system into small components.<br>
2. Each service has its own database. Use cache heavily.<br>
3. The service is sharded.<br>
4. The services talk to each other asynchronously through message queues.<br>
5. The service is implemented using Event Sourcing with CQRS.<br>
6. Showing off knowledge in distributed systems such as eventual consistency, CAP theorem, etc.<br>

𝐖𝐡𝐚𝐭 𝐢𝐭 𝐚𝐜𝐭𝐮𝐚𝐥𝐥𝐲 𝐢𝐬 <br>
Stack Overflow serves all the traffic with only 9 on-premise web servers, and it’s on monolith! It has its own servers and does not run on the cloud.<br>
