# The Rise of Microservices

Large companies, especially those running massive online services, were looking for better ways to handle growth and reliability. 
Thankfully, many early adopters of microservices began sharing their experiences and lessons learned.

At first, most of these companies used monolithic applications—large systems where all parts of the software were tightly connected. 
This worked well in the beginning and often helped them grow fast from a business point of view.

However, as these systems became bigger, they also became harder to maintain, update, and scale. Even the most powerful servers could not keep up with the demand. This limitation, known as vertical scaling, means trying to make a single machine stronger and faster, which has limits.

To solve this, developers started to split their applications into smaller, independent parts. Each part could be deployed, updated, and scaled on its own. This new method is called horizontal scaling, where you run several smaller copies of a service on different servers. A load balancer then distributes user requests among them.

In the cloud, this scaling can seem almost unlimited—you can add more virtual servers whenever you need, as long as your software is designed to handle many copies.

During that time, several open-source tools appeared, making microservices easier to build and manage. Let’s look at some key examples:

- `Spring Cloud` simplified complex features like service discovery, configuration management, and monitoring, helping developers build distributed systems faster.
- `Docker` introduced containers, allowing developers to package applications with all their dependencies. 
    This made it much easier to run the same software on any computer—from a developer’s laptop to a cloud server.
- But running containers manually isn’t enough for production. You also need systems that make sure containers stay online, restart when they fail, and scale when needed.    
- This led to the creation of container orchestrators such as Apache Mesos, Docker Swarm, Amazon ECS, HashiCorp Nomad, and especially Kubernetes.
Kubernetes, created by Google and released in 2015, quickly became the standard. It was later donated to the `Cloud Native Computing Foundation` (`CNCF`). Today, almost every cloud provider supports it.
- A few years later, a new idea called a service mesh appeared. A service mesh adds extra tools that help microservices communicate securely and reliably. It works together with an orchestrator like Kubernetes, reducing the amount of code developers need to manage network concerns such as retries, security, and load balancing.

## What Exactly Is a Microservice?

A microservice architecture breaks a large application into a set of smaller, independent services. This design has two main benefits: