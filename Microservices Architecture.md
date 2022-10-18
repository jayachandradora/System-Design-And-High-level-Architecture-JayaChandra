# System-Design-And-High-level-Architecture-JayaChandra
Software Design Architecture - High Level Design

![image](https://user-images.githubusercontent.com/115500959/195155870-57b0f898-0f4d-4e44-9cf3-e4a29ebe4905.png)

### Details of the Components
What does a typical microservices architecture look like? And when should we use it? Let’s take a look.<br>

Microservices are loosely coupled. Each service handles a dedicated function inside a large-scale application. For example, shopping cart, billing, user profile, and push notifications can all be individual microservices. <br>

The diagram below shows a typical microservice architecture. <br>

🔹Load Balancer: This distributes incoming traffic across multiple backend services. <br>

🔹CDN (Content Delivery Network): CDN is a group of geographically distributed servers that hold static content for faster delivery. The clients look for content in CDN first, then progress to backend services.<br>

🔹API Gateway: This handles incoming requests and routes them to the relevant services. It talks to the identity provider and service discovery.<br>

🔹Identity Provider: This handles authentication and authorization for users. <br>

🔹Service Registry & Discovery: Microservice registration and discovery happen in this component, and the API gateway looks for relevant services in this component to talk to. <br>

🔹Management: This component is responsible for monitoring the services.<br>

🔹Microservices: Microservices are designed and deployed in different domains. Each domain has its own database. The API gateway talks to the microservices via REST API or other protocols, and the microservices within the same domain talk to each other using RPC (Remote Procedure Call).<br>

Benefits of microservices:<br>
- They can be quickly designed, deployed, and horizontally scaled.<br>
- Each domain can be independently maintained by a dedicated team.<br>
- Business requirements can be customized in each domain and better supported, as a result.<br>

Over to you: 1). What are the drawbacks of the microservice architecture?<br>
2). Have you seen a monolithic system be transformed into microservice architecture? How long does it take?<br>
