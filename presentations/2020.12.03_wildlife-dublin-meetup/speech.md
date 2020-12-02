## How Wildlife Studios built a Global Multi Cluster Gaming Infrastructure with Cilium

### Self presentation

Hello, my name is Luan and I've been working at Wildlife for the past two years in the SRE team maintaining a global gaming infrastructure using Kubernetes, Cilium and other tools that supports millions of daily active users playing our games and using our services. Today I'm here, in this 30 minutes talk, to show you some reasons why we have chosen Kubernetes and Cilium to build this whole infrastructure and how it handles thousands of game components distributed all over the world.

[SLIDE]

### Introduction

The story I want to tell started a couple of time ago with a few amazing engineers taking care of almost everything that was not frontend stuff, creating services, crimpping cables, fixing computers, and also dealing with engineering and infrastructure problems for all the company. At that time we already had huge global games like Sniper3D and Bikerace. This team were called only backend, and we can say they were much more than brilliant engineers, but visioneers that took really good decisions at that time which created the path for a solid evolution of this global gamming architecture. Some of them: early adption of distributed systems, linux, micro-services, containers, monitoring stacks, and of course Kubernetes.

So, things started growing fast, everyday was new day with new challenges. There was no other way, we created an infrastructure team, totally dedicated to this. The goal was to make it possible to grow the infrastructure as fast as the other parts of the company, we shouldn't be blockers. The beggining of this new team brings us to the end of 2018, when we started the preparations for two big game launches planned to be released in the middle of 2019. Tennis Clash and Zooba. I don't know if you already played them, they are awesome. But back to the content, we were thinking high! Our metrics indicated these games were supposed to be really big, and after the release we confirmed this. For example Tennis was the most downloaded game in more than 100 countries. So, months before the launches we had already started to prepare everything, profiling, understanding bottlenecks, proposing solutions and thinking how we could improve our infrastructure to deal with a bunch of scalability, reliability and visibility challenges for the future.

From the technical perspective, one of the most important things to do was to rethink our Kubernetes networking model in order to support better integration between workloads in different AWS regions, which was really important for our micro-services. 

At that time, the CNI plugin that we were using didn't support pod-to-pod communication, so we were exposing services through internal load balancers and connecting parts of our internal game architecture using NATS.

To understand why we needed pod-to-pod communication and this better integration between different regions, it's necessary to look at the previews architecture and how our games relies on some technologies we built here. So, I'm gonna show you two different systems we use for creating our games.

[SLIDE]

The first one is called "Pitaya". Pitaya a lightweight game server framework used to build several components of each game. It creates an efficient communication layer that integrates several parts of a game. But what are these parts?

So, the basic skeleton of a game has three parts. Lets talk about them: the connector, the metagame and the game room.

The connector is where the player connects to the game, it's responsible for checking the user identity and also for handling the session holding the TCP connection open. Since the user is connected, the device can interact with the game. Here, the connector redirects the packets to an available metagame.

Metagame handles the business logic, the user information, gems, trophies, points, coins, everything that isn't the gameplay itself. It communicates with other services for gathering external information, sharing events, validating receipts and asking for available game rooms when the user wants to join a room to play.

When a device asks for joining in a room, the metagame checks the available rooms in a matchmaker server. On the matchmaker side, amazing algorithms select players based on their levels, skills, other inputs and put them together. Finally the matchmaker server returns the correspoding public endpoint of this room, where these similar users can play together. 

In the end, the game room is where the fun happens. It has all the gameplay logic. It connects different players and bots and has the rules that makes the experience of playing online games amazing. Users send UDP packets directly to the game room regardless the connector, after all this is a critical latency point. For some games 100ms latency is likely a bad gaming experience. So things should be fast, we will come back in this discussion later.

Now I have a good question: how do we connect these different parts: connector, metagame and game rooms?

Pitaya peers use a centralized Etcd cluster for auto-discovery. So, when a new peer appears, it registers itself on this dedicated etcd cluster and also fetches the list of peers already registered. Therefore they can communicate with each other in two different ways, through NATS topics, or a more performatic method, relying on gRPC and talking directly to each other.

Great, now that you already know about Pitaya, lets talk about the second important system.

[SLIDE]
 
It is called "Maestro", a game room scheduler for Kubernetes, which is responsible for scaling up and down different stacks of game rooms on demand, making sure that we always have game rooms available for new matches, and of course if we're not using more rooms than the necessary. Yeah, saving resources. Maestro talks to the Kubernetes API to create and destroy game rooms with public endpoints that players can connect.

Each Kubernetes cluster has its own Maestro stack that controls all the game rooms lifecycle.

But I said that the Matchmaker server was responsible for returning the available gamerooms for metagame when a player asks to play. How it works? Maestro has a forward mechanism for gameroom events, for example, when a new gameroom is registered, maestro forwards this event to the matchmaker server. Matchmaker saves this entry on its own pool of rooms, and control this based on those forwarded events. That's how matchmaker knows which game room is available when the metagame asks for a place to the player.

[SLIDE]

This is a really quick overview of a simple internal architecture of games, of course there are a bunch of other components involved, but this is what we need to start talking how to make online games possible in a global scale. And, you know, talking about global scale is the same as talking about global problems.

So, here we go with a big global problem! Latency. Simple. In the world of games latency is critical. And we'll start adding more complexity to the architecture I presented because of this. In order to reduce latency and improve the user experience. That's why our game room stacks are deployed in several regions around the world. We have several Kubernetes clusters running Maestro, controlling thousands of game rooms in different regions, North America, Europe, Asia and so on. This also creates a fault-tolerant system, where it's possible to turn off game rooms in specific regions and redirect users to another point of the globe for a while, during a maintaince period, a troubleshooting, or something like this.

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
