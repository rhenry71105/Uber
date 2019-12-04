# Uber
Uber's Story

IS 362 Week 14 Discussion Recommender 

Introduction

Uber's story starts out simple. Uber’s owners have an idea for an app. One of those game-changing, “how has nobody thought of this!”, kind of ideas (or at least that's what we all like the think they are at first). The idea was to create a service that would aggregate the current price and time estimates for transportation service and give users the ability to select whichever was cheapest or fastest, depending on what you had requested. Simple enough right?
Because this post is supposed to be about reverse-engineering the functionality behind Uber's app (mainly the use of their private API endpoints) I will try to paint you a full picture of the technology and services utilized by Uber. From the start, Uber’s mission is transportation as reliable as running water, everywhere, for everyone. To make that possible, Uber needed to build, create and work with complex data. Then They bundle it up neatly as a platform that enables drivers to get business and riders to get around.
While Uber’s UI seems to be simple, it is engineered with complex systems behind it to stay up, handle difficult interactions, and serve massive amounts of traffic. It’s broken up from the original monolithic architecture into many parts to scale with growth. With hundreds of microservices that depend on each other, drawing a diagram of how Uber works at this point is wildly complicated, and it all changes rapidly. What can be cover in a two-part paragraph is the stack used by Uber.
Uber Engineering’s Challenges: Hypergrowth.
As Uber faces the same global-scale problems as some of the most successful software companies, but they're only six years old, so they haven’t solved them yet, and their business is based in the physical world in real-time.
Uber’s 300 operational cities were scattered across the map.
Unlike freemium services, Uber has only transactional users: riders, drivers, and now eaters and couriers. People rely on Uber’s technology—to make money, to go where they need to go—so there’s no safe time to pause. Uber will need to prioritize availability and scalability.
As Uber expands on the roads, their service must scale. Uber’s stack’s flexibility encourages competition so the best ideas can win. These ideas aren’t necessarily unique. If a strong tool exists, it’s utilized until Uber’s needs exceed its abilities. When Uber needs something more, It’s built in-house by Uber solutions. Uber Engineering has responded to growth with tremendous adaptability, creativity, and discipline in the past year. Throughout 2019, Uber have even bigger plans. By the time you read this, much will have changed, but this is a snapshot of what Uber is using now. Through Uber descriptions, they have demonstrated their philosophy around using tools and technologies.

Uber’s Tech Stack

Instead of a tower of restrictions, picture a tree. Looking at the technologies across Uber, you see a common stack (like a tree trunk) with different emphases for each team or engineering office (its branches). It’s all made of the same stuff, but tools and services bloom differently in various areas.
Bottom: Platform
This first article focuses on the Uber platform, meaning everything that powers the broader Uber Engineering organization. Platform teams create and maintain things that enable other engineers to build programs, features, and apps used by consumers.

Infrastructure and Storage

Uber business runs on a hybrid cloud model, using a mix of cloud providers and multiple active data centers. If one data center fails, trips (and all the services associated with trips) failover to another one. Uber assigns cities to the geographically closest data center, but every city is backed up on a different data center in another location. This means that all of Uber data centers are running trips at all times; They have no notion of a “backup” data center. To provision this infrastructure, Uber uses a mix of internal tools and Terraform.
Uber's needs for storage have changed with growth. A single Postgres instance got Uber through their infancy, but Uber grew so quickly, Uber needed to increase available disk storage and decrease system response times.
Uber started Project Mezzanine refactored the system to match this high-level architecture.

Uber currently use Schemaless (built in-house on top of MySQL), Riak, and Cassandra. Schemaless is for long-term data storage; Riak and Cassandra meet high-availability, low-latency demands. Over time, Schemaless instances replace individual MySQL and Postgres instances, and Cassandra replaces Riak for speed and performance. For distributed storage and analytics for complex data, Uber uses a Hadoop warehouse. Beyond these databases, Uber focus on building a new real-time data platform.
Uber uses Redis for both caching and queuing. Twemproxy provides scalability of the caching layer without sacrificing the cache hit rate via its consistent hashing algorithm. Celery workers process async workflow operations using those Redis instances.
Uber services interact with each other and mobile devices, and those interactions are valuable for internal uses like debugging as well as business cases like dynamic pricing. For logging, Uber uses multiple Kafka clusters, and the data is archived into Hadoop and/or a file storage web service before it expires from Kafka. This data is also ingested in real-time by various services and indexed into an ELK stack for searching and visualizations (ELK stands for Elasticsearch, Logstash, and Kibana).
Uber uses Docker containers on Mesos to run their microservices with consistent configurations scalable, with help from Aurora for long-running services and cron jobs. One of Uber infrastructure teams, Application Platform, produced a template library that builds services into shippable Docker images.

Uber services

Uber service-oriented architecture (SOA) makes service discovery and routing crucial to Uber’s success. Services must be able to communicate with each other in a complex network. Uber has used a combination of HAProxy and Hyperbahn to solve this problem. Hyperbahn is part of a collection of open-source software developed at Uber: Ringpop, TChannel, and Hyperbahn all share a common mission to add automation, intelligence, and performance to a network of services.
Legacy services use local HAProxy instances to route JSON over HTTP requests to other services, with front-end web server NGINX proxying to servers in the back end. This well-established way of transferring data makes troubleshooting easy, which was crucial throughout several migrations to newly developed Uber’s systems.
However, Uber is prioritizing long-term reliability over debuggability. Alternative protocols to HTTP (like SPDY, HTTP/2, and TChannel) along with interface definition languages like Thrift and Protobuf will help evolve their system in terms of speed and reliability. Ringpop, a consistent hashing layer, brings cooperation and self-healing to the application level. Hyperbahn enables services to find and communicate with others simply and reliably, even as services are scheduled dynamically with Mesos.
Instead of archaically polling to see if something has changed, Uber is moving to a pub-sub pattern (publishing updates to subscribers). HTTP/2 and SPDY more easily enable this push model. Several poll-based features within the Uber app will see a tremendous speed up by moving to push.

Development and Deploy

Phabricator powers a lot of internal operations, from code review to documentation to process automation. Engineers search through code on OpenGrok. For Uber’s open-source projects, Uber develop in the open using GitHub for issue tracking and code reviews.
Uber strives to make development simulate production as closely as possible, so they are developed mostly on virtual machines running on a cloud provider or a developer’s laptop. Uber built their own internal deployment system to manage builds. Jenkins does continuous integration. Uber combined Packer, Vagrant, Boto, and Unison to create tools for building, managing, and developing on virtual machines. Uber uses Clusto for inventory management in development. Puppet manages system configuration.
Uber is constantly working to build and maintain stable communication channels, not just for their services but also for their engineers. For information discovery, Uber built uBlame (a nod to git-blame) to keep track of which team owns a particular service, and Whober for looking up names, faces, contact information, and organizational structure. Uber uses an in-house documentation site that auto builds docs from repositories using Sphinx. An enterprise alerting service alerts our on-call engineers to keep systems running.  Most developers run OSX on their laptops, and most of our production instances run Linux with Debian Jessie.

Languages

At the lower levels, Uber’s engineers primarily write in Python, Node.js, Go, and Java. Uber started with two main languages: Node.js for the Marketplace team, and Python for everyone else. These first languages still power most services running at Uber today.
Uber adopted Go and Java for high-performance reasons. They provide first-class support for these languages. Java takes advantage of the open-source ecosystem and integrates with external technologies, like Hadoop and other analytics tools. Go gives Uber efficiency, simplicity, and runtime speed.
Uber ripped out and replace older Python code as they break up the original codebase into microservices. An asynchronous programming model gives Uber better throughput. Uber uses Tornado with Python, but Go’s native support for concurrency is ideal for most new performance-critical services.
Uber write tools in C and C++ when it’s necessary (like for high-efficiency, high-speed code at the system level). Uber uses software that’s written in those languages—HAProxy, for example—but for the most part, Uber doesn’t actually work in them.
And, of course, those working at the top of the stack write in languages beyond Java, Go, Python, and Node.

Testing

To make sure that Uber services can handle the demands of their production environment, they developed two internal tools: Hailstorm and uDestroy. Hailstorm drives integration tests and simulates peak load during off-peak times, while uDestroy intentionally breaks things so they can get better at handling unexpected failures.
Uber employees use a beta version of the app to continuously test new developments before they reach users. Uber made an app feedback reporter to catch any bugs before they roll out to users. Whenever testers take a screenshot in the Uber apps, this feature prompts Uber to file a bug-fix task in Phabricator.

Reliability

Uber engineers that write backend services are responsible for their operations. If they write some code that breaks in production, they get paged. They use Nagios alerting for monitoring, tied to an alerting system for notifications.
Aiming for the best availability and 1 billion rides per day, site reliability engineers focus on getting services that they need to succeed.
