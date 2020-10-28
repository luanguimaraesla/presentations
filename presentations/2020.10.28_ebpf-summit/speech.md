## How Wildlife Studios built a Global Multi Cluster Gaming Infrastructure with Cilium

### Self presentation

Hello, my name is Luan and I've been working at Wildlife for the past two years maintaining a global gaming infrastructure using Kubernetes, Cilium and other amazing tools that supports millions of daily active users playing our online games. And I'm here to talk about why we have chosen Cilium and eBPF and how it has become one of the most important parts of our architecture.

[SLIDE]

### Introduction

In the end of 2018 we started the preparations for two big game launches planned to be released in the middle of 2019. One of the most important things to do was to rethink our Kubernetes networking model in order to support better integration between workloads in different AWS regions, which was really important for our micro-services. 

The CNI plugin that we were using didn't support pod-to-pod communication at that time, so we were exposing services through internal load balancers and connecting parts of our internal game architecture using NATS.

To understand why we needed pod-to-pod communication, I'm gonna show you two different systems we use for creating our games.

[SLIDE]

The first one is called "Pitaya". Pitaya a lightweight game server framework used to build several components of each game. These components use a centralized Etcd cluster for auto-discovery, therefore they can communicate with each other in two different ways, through NATS topics, or a more performatic method, relying on gRPC.

[SLIDE]

And the other system is called "Maestro", a game room scheduler for Kubernetes, which is responsible for scaling up and down different stacks of game rooms on demand, making sure that we always have game rooms available for new matches.

[SLIDE]

In order to reduce latency and improve the user experience, our stacks are deployed in several regions around the world. This also creates a fault-tolerant system, where it's possible to turn off game rooms in specific regions and redirect users to another point of the globe for a while.

### Problems

So, lets talk about our old infrastructure and problems we had. It was really hard to configure routing for pod-to-pod communication in a easy way in order to enable Pitaya’s gRPC integration in multiple clusters. It was also hard in terms of scalability, because for each shared service, like Etcd, NATS, Jaeger, and so on, we had to maintain internal load balancers and their respective DNS records. And we were of course looking for a tool that could improve the visibility of the network without adding tons of specialized components to our clusters.

The big question: How can we create and monitor a highly available global networking for critical game components that need to reach each other and also communicate with some centralized shared services?

### Tests and Results

As the existing clusters were deployed using Kops, some different supported CNI plugins appeared as good candidates to meet the requirements. So we ran several tests against the available options, comparing from the setup complexity to the networking performance. After analysing Cilium's positive results, like the stability running hundreds of nodes, and the small amount of requests to the Kubernetes API we came up with a plan to rollout Cilium for all the production clusters. 

[SLIDE]

The migration process was really challenging and it took months of work. Thanks to the good documentation and the amazing support of the Cilium community, we could easily configure the Cluster Mesh feature, making it possible to enable the gRPC mode on Pitaya. The pod-to-pod communication improved the servers' performance and removed one of those single points of failure - NATS.

In a second migration round, we replaced most of the internal load balancers we had for Cilium Global Services. This process included the migration of those load balancers attached to Pitaya's Etcd and to internal logging and tracing stacks. This removed the complexity of maintaining a lot of cloud-scoped load balancers for internal usage and their respective private DNS records.

This is how it looks after the migration. The game rooms and other game components talk to each other using Cluster Mesh vxlan layer and here you can see the Global Service to reach Etcd.

### Final

Today, we have Cilium deployed in almost 30 Kubernetes production clusters. Each game has at least 3 clusters running together in the same Cluster Mesh configuration. This infrastructure handles more than 50 (fifty) thousand client requests per second and supports millions of daily active users.

Since the migration we've been working with the Cilium team to improve our systems. This close interaction supported some investigations and allowed us to solve problems together, increasing the reliability of our network.

Now we're looking forward to improving the security of our environment with the powers of Network Policies, towards the Zero Trust Network initiative. 

Another promising tool is Hubble, that we already tested in a few clusters. Now we're very excited to rollout this tool for all the production environments to obtain detailed insights about our network with minimum overhead, thanks to the use of eBPF.

So this is how we're using Cilium at Wildlife. And If you want to know more about the story behind this journey, you can check the post I published on the Cilium blog on September 3rd detailing everything about it.

Thanks! Hope you enjoyed, folks.
