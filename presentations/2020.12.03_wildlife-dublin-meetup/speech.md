[SLIDE - 1]

## How Wildlife Studios built a Global Multi Cluster Gaming Infrastructure with Cilium

### Self presentation

Hello, my name is Luan and I've been working at Wildlife for the past two years in the SRE team maintaining a global gaming infrastructure using Kubernetes, Cilium and other tools that supports millions of daily active users playing our games and using our services. Today I'm here, in this 30 minutes talk, to show you some reasons why we have chosen Kubernetes and Cilium to build this whole infrastructure and how it handles thousands of game components distributed all over the world.

[SLIDE - 2]

### Introduction

The story I want to tell started a couple of years ago with a few amazing engineers taking care of almost everything that was not frontend systems, creating backend services, crimpping cables, fixing computers, and also dealing with engineering and infrastructure problems for all the company. At that time we already had huge global games like Sniper3D and Bikerace. And this team were called only backend, and we can say they were much more than brilliant engineers, but visioneers that took really good decisions at that time which created the path for a solid evolution of this global gamming architecture. Some of them are: early adption of distributed systems, linux, micro-services, containers, monitoring stacks, and of course Kubernetes.

[SLIDE - 3]

So, things started growing fast, everyday was new day with new challenges. And of course, there was no other way, we created an infrastructure team, totally dedicated to this. The goal was to make it possible to grow the infrastructure as fast as possible, and as fast as the other parts of the company, we shouldn't be blockers and our systems should have an excellent quality.

[SLIDE - 4]

The beggining of this TEAM brings us to the end of 2018, when we started the preparations for two big game launches planned to be released in the middle of 2019. Tennis Clash and Zooba. I don't know if you already played them, they are awesome. But back to the content, we were thinking high! Our metrics indicated these games were supposed to be really big, and after the release we confirmed this. For example Tennis was the most downloaded game in more than 100 countries. So, months before the launches we had already started to prepare everything, profiling, understanding bottlenecks, proposing solutions and thinking how we could improve our infrastructure to deal with a bunch of scalability, reliability and visibility challenges for the future.

[SLIDE - 5]

From the technical perspective, one of the most important things to do was to rethink our Kubernetes networking model in order to support better integration between workloads in different AWS regions, which was really important for our micro-services. 

At that time, the CNI plugin that we were using didn't support pod-to-pod communication, so we were exposing services through internal load balancers and connecting parts of our internal game architecture using NATS.

To understand why we needed pod-to-pod communication and this better integration between different REGIONS, it's necessary to look at the previews architecture and how our games relies on some technologies we built here. So, I'm gonna show you two different systems we use for creating our games.

[SLIDE - 6]

The first one is called "Pitaya". Pitaya is a lightweight game server framework used to build several components of each game. It creates an efficient communication layer that integrates several parts of a game. But what are these parts?

[SLIDE - 7] 

So, the basic skeleton of a game has three parts. Lets talk about them: the connector, the metagame and the game room.

[SLIDE - 8]

The connector is where the player connects to the game, it's responsible for selecting the correct stack the player should use and also for handling the session holding the TCP connection. Since the user is connected, the device can start the interaction with the game. Here, the connector redirects the packets to an available metagame through Pitaya's communication layer.

[SLIDE - 8]

Metagame handles the business logic, the user information, gems, trophies, points, coins, everything that isn't the gameplay itself. It communicates with other services for gathering external information, sharing events, validating receipts and asking for available game rooms when the user wants to join a room to play.

When a device asks for joining in a room, the metagame requests an available room to a matchmaker server. On the matchmaker side, matching algorithms select players based on their levels, skills, other inputs and put them together. Finally the matchmaker server returns the correspoding public endpoint of this room, where these similar users can play together. 
  
[SLIDE - 9]

In the end, the game room is where the fun happens. It has all the gameplay logic. It connects different players and bots and has the rules that make the experience of playing online games amazing. Users send UDP packets directly to the game room regardless the connector, after all this is a critical latency point. For some games 100ms latency is likely a bad gaming experience. So things should be really fast! But, we're going back to this topic later.

Now I have a good question: how do we connect these different parts: connector, metagame and game rooms?

[SLIDE - 10]

Pitaya peers use a centralized Etcd cluster for auto-discovery. So, when a new peer appears, it REgisters itself on this dedicated etcd cluster and also fetches the list of peers already registered. Therefore they can communicate with each other in two different ways, through NATS topics, or a more performatic method, relying on gRPC and talking directly to each other.

Great, now that you already know about Pitaya, lets talk about the second important system.

[SLIDE - 11]
 
It is called "Maestro". Maestro is a game room scheduler for Kubernetes, which is responsible for scaling up and down different stacks of game rooms on demand, making sure that we always have game rooms available for new matches, and of course if we're not using more rooms than the necessary. Maestro talks to the Kubernetes API to create and destroy game rooms with public endpoints that players can connect.

Each Kubernetes cluster has its own deployment of Maestro that controls all the game rooms lifecycle inside a single cluster.

[SLIDE - 13]

But I said the Matchmaker server was responsible for returning the available game room for the metagame when a player asks to play.

But how does it work? Maestro has a forward mechanism for gameroom EVENTS, for example, when a new gameroom is registered, maestro forwards this EVENT to the matchmaker server. Matchmaker stores this game room information on its own pool of rooms, and then control this based on other forwarded events. That's how matchmaker knows which game room is available when the metagame asks for a place for a player.

What I said was a really quick overview of a simple internal architecture of games we use here, of course there are a bunch of other components involved, but this is all we need to start talking about how to make online games possible in a global scale. And, you know, talking about global scale is the same as talking about global problems.

So, here is a big global problem! Latency. 

[SLIDE - 14]

In the world of games latency is critical. And we'll start adding more complexity to the architecture I PRESENTED because of this. In order to reduce latency and improve the user experience. That's why our game rooms are spread in several regions around the world. It would be nice and SIMPLER if it could be deployed in just a single place, but that's not the situation.

We have several Kubernetes clusters running Maestro, controlling thousands of game rooms in different regions, North America, Europe, Asia and so on. This also creates a fault-tolerant system, where it's possible to turn off game rooms in specific regions and redirect users to another point of the globe for a while, during a maintaince period, a troubleshooting, or something like this.

[SLIDE - 15]

So, lets talk about our old infrastructure and problems we had.

[SLIDE - 16]

### Problems

 It was really hard to configure routing for pod-to-pod communication in a easy way in order to enable Pitaya’s gRPC integration in multiple clusters. It was also hard in terms of scalability, because for each shared service, like Etcd, NATS, Jaeger, and so on, we had to maintain internal load balancers and their respective DNS records. And also, we were of course looking for a tool that could improve the visibility of the network without adding tons of specialized components to our clusters.

[SLIDE - 18]

The big question was: How can we create and monitor a highly available global networking for critical game components that need to reach each other and also communicate with some centralized shared services?

[SLIDE - 19]

These are some important requirements we were lookin for:

* Low-latency pod-to-pod communication across different clusters
* Global service network without dramatically increasing system complexity adding a bunch of tools, controllers, and resources
* Reasonable use of resources, because of the criticality of adding a new daemonset to the system
* Reliable scaling up to a 600 node in a single cluster
* Boot quickly enough to withstand a super dynamic game-room environment
* Simple configuration in order to reduce the operational cost
* A good set of metrics that could be exported to Prometheus and Datadog
* Good documentation and a nice community support

As the existing clusters were deployed using Kops, some different supported CNI plugins appeared as good candidates to fill the requirements. So we ran several tests against the available options, comparing from the setup complexity to the networking performance.

I would like to give special attention to the part of simple configuration, because it was a hard problem to solve. Just for comparison, CONFIGURING Calico 2.6 to support MULTÍ-cluster communication was really painful, as it didn’t have a standard way to use a reliable HA route reflector for exchanging routes. It meant that we needed to use its normal BGP full-node-mesh mode, but in this way, it would be virtually impossible to deploy a cluster with more than 300 nodes.

[SLIDE - Calico 20]

But we tried! And you're looking at the result. We decided to create an external BIRD controller, building a full-mesh of route reflectors. It worked but we were 100% sure that we needed another solution. I need to say that today things changed, this setup was configured almost 3 years ago, Calico project evolved so fast and it has a lot of nice features, including support to eBPF, so don't take these results as true for now.

[SLIDE - 21]

### Tests and Results

The final report was clear, but we were a little hesitant to adopt such a new technology. We thought about our knowledge in troubleshooting iptables, the team was small, there were just a few people talking about Cilium and eBPF, and all those feelings before a big change. But there's a law here: after the research, if the data is clear, we go for it. The report was pretty much sharp, and after analysing Cilium's positive results, like the stability running hundreds of nodes, the small amount of requests to the Kubernetes API, the performatic way it deals with UDP traffic, and so on, we came up with a plan to rollout Cilium for all the production clusters. 

[SLIDE - 22]

The migration process was really challenging and it took months of work. But everything worked! Thanks to the good documentation and the amazing support of the Cilium community. I won't say it was a path without problems, specially because of the necessity of some Cilium's bleeding edge features, but we were working very close to them, reporting problems, performing load tests, running new features in production.  It was a really nice opportunity to learn about eBPF, Cilium and of course Kubernetes internals.

[SLIDE - 23]

Nice, lets go with the new infrastructure and Cilium!

[SLIDE - 24]

After this whole migration process, with a good understand of how cilium worked, we configured the ClusterMesh feature, making it possible to enable the gRPC mode on Pitaya. The pod-to-pod communication improved the servers' performance and removed one of those single points of failure - which was NATS. The point is that the problem wasn't NATS itself, NATS is an amazing project and we're still using this for some games with special requirements. But we don't need this in a number of use cases, that's why we wanted to change things.

In a second migration round, we replaced most of the internal load balancers we had for Cilium Global Services. This process included the migration of those load balancers attached to Pitaya's Etcd and to internal logging and tracing stacks. This removed the complexity of maintaining a lot of cloud-scoped load balancers for internal usage and their respective private DNS records.

Finally, this is how it looks after the migration. The game rooms and other game components talk to each other using Cluster Mesh vxlan layer and here you can see the Global Service to reach Etcd. Actually we use this for a bunch of other services like Matchmaker and the logging stack for example.


### Clustermesh and Global Services

But what are these two amazing features we're talking about: Clustermesh and Global Services.

I'll show you these two features in a quick demo. I will also take this opportunity to PRESENT how it's possible to run, and keep several kubernetes clusters updated with a minimum overhead.

[SLIDE 25 - GO TO TERMINAL]

* create an empty folder
* run bigbang init .
* run bigbang create cluster p1.us-east-1.test.tfgco.com
* run bigbang create cluster p2.eu-central-1.test.tfgco.com
* run bigbang create clustermesh presentation
* apply cluster changes
* apply clustermesh changes
* wait for them to be running
* run bigbang create addon cilium for both clusters
* apply cilium
* helm template and apply your deployments
* show cilium CLI
* apply the clustermesh configuration
* restart cilium
* show global services and clustermesh configuration working

[BACK TO THE PRESENTATION]

Hope everything is clear so far, there were a bunch commands and concepts together but feel free to ask anything in the end of the presentation if you have doubts. 

[SLIDE - 26]

### How it works globally

So imagine you're trying to play one of our games from Dublin, in this example the REGION B:

First imagine that you connect to the metagame in the REGION A, because it's not latency sensitive, so it could be a little bit far from the users.
Then you check your points, talk to your friends and decide to play a match.

Based on a ping system, your device already knows which region is better for you to play according to the latency.
So you ask a new room to the metagame, that asks an available room to the matchmaker server.

Matchmaker server has a pool of available endpoints it got from Maestro forwarded events and it knows you're a player from Europe.
You receive a public endpoint from the metagame and then connect directly to this endpoint to play with people around you.

This is the big picture of the game.

[SLIDE - 27]

### Final

Today, we have Cilium deployed in more than 30 Kubernetes production clusters. Each game has at least 3 clusters running together in the same Cluster Mesh configuration. This infrastructure handles more than 50 (fifty) thousand client requests per second and supports millions of daily active users.

Since 2018 we've been proactively working with the Cilium team to improve our systems. As I said, this close interaction supported some investigations and allowed us to efficiently solve problems together, increasing the reliability of our network.

After this long period migrating and building tools around Cilium, we're looking forward to improving the security of our environment with the powers of Network Policies. We can say this is a step towards the Zero Trust Network strategic initiative.

Another PROMISING step is related to Hubble, a distributed observability platform built on top of Cilium for cloud native workloads. We're very excited after rolling out this tool for all the production environments in order to obtain detailed insights about our network, specially about packet drop and traffic.

So this is how we're using Kubernetes and Cilium at Wildlife to build a global gaming infrastructure. And If you want to know more about the story behind this journey or about the internals of this archtecture, you can check the post I published on the Cilium blog on September 3rd detailing everything about it and also Wildlife's tech blog. There are some good posts about the backstage tools for building games, and I recommend a post about Pitaya my friend Camila Scatolini wrote there.

[SLIDE - 28]

Thanks! Hope you enjoyed. It was a really good time. Thanks everybody :)
