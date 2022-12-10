# Netflix API architecture

The Netflix API architecture went through 4 main stages. 

𝐌𝐨𝐧𝐨𝐥𝐢𝐭𝐡. The application is packaged and deployed as a monolith, such as a single Java WAR file, Rails app, etc. Most startups begin with a monolith architecture.

𝐃𝐢𝐫𝐞𝐜𝐭 𝐚𝐜𝐜𝐞𝐬𝐬. In this architecture, a client app can make requests directly to the microservices. With hundreds or even thousands of microservices, exposing all of them to clients is not ideal.

𝐆𝐚𝐭𝐞𝐰𝐚𝐲 𝐚𝐠𝐠𝐫𝐞𝐠𝐚𝐭𝐢𝐨𝐧 𝐥𝐚𝐲𝐞𝐫. Some use cases may span multiple services, we need a gateway aggregation layer. Imagine the Netflix app needs 3 APIs  (movie, production, talent) to render the frontend. The gateway aggregation layer makes it possible.

𝐅𝐞𝐝𝐞𝐫𝐚𝐭𝐞𝐝 𝐠𝐚𝐭𝐞𝐰𝐚𝐲. As the number of developers grew and domain complexity increased, developing the API aggregation layer became increasingly harder. GraphQL federation allows Netflix to set up a single GraphQL gateway that fetches data from all the other APIs.

![image](https://user-images.githubusercontent.com/115500959/206849624-09068de7-a7c3-47be-878e-e6c95d2d81a7.png)

