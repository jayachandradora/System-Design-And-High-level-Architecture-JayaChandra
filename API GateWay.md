# System-Design-And-High-level-Architecture-JayaChandra
Software Design Architecture - High Level Design

![image](https://user-images.githubusercontent.com/115500959/196360977-34d3b579-a89b-404a-99b1-61f3068d8a00.png)

### What does API gateway do?

The diagram below shows the detail.<br>

Step 1 - The client sends an HTTP request to the API gateway.<br>

Step 2 - The API gateway parses and validates the attributes in the HTTP request.<br>

Step 3 - The API gateway performs allow-list/deny-list checks.<br>

Step 4 - The API gateway talks to an identity provider for authentication and authorization.<br>

Step 5 - The rate limiting rules are applied to the request. If it is over the limit, the request is rejected.<br>

Steps 6 and 7 - Now that the request has passed basic checks, the API gateway finds the relevant service to route to by path matching.<br>

Step 8 - The API gateway transforms the request into the appropriate protocol and sends it to backend microservices.<br>

Steps 9-12: The API gateway can handle errors properly, and deals with faults if the error takes a longer time to recover (circuit break). 
It can also leverage ELK (Elastic-Logstash-Kibana) stack for logging and monitoring. We sometimes cache data in the API gateway.<br>
