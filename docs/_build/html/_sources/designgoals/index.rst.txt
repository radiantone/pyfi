Goals
================================
.. toctree::
   :maxdepth: 2

As the name suggests, ElasticCode is a spiritual offshoot of Apache NIFI except built using a python stack for running python (and other scripting languages) processors.
However, ElasticCode is designed to be more broad in terms of design and scope which we will discuss below.

Some important design goals for this technology are:

1. **Fault-Tolerant** - ElasticCode runs as a distributed network of logical compute processors that have redundancy and load-balancing built in.
2. **At-Scale** - This phrase is important. It indicates that the logical constructs (e.g. pyfi processors) run at the scale of the hardware (e.g. CPU processors), meaning there is a 1-1 correlation (physical mapping) between hardware processors and pyfi processors.
3. **Secure** - All the functional components in ElasticCode (database, broker, storage, cache) have security built in.
4. **Dynamic** - The topology and behavior of a ElasticCode network can be adjusted and administered in real-time without taking down the entire network. Because ElasticCode is not a single VM controlling everything, you can add/remove update components without negatively impacting the functionality of the system.
5. **Distributed** - As was mentioned above, everything in ElasticCode is inherently distributed, down to the processors. There is no physical centralization of any kind.
6. **Performance** - ElasticCode is built on mature technology stack that is capable of high-throughput message traffic.
7. **Reliability** - The distributed queue paradigm used by ElasticCode allows for every processor in your dataflow to consume and acknowledge message traffic from its inbound queues and write to outbound queues. These durable queues persist while processors consume messages off them.
8. **Scalability** - Processors can scale across CPUs, Machines and networks, consuming message traffic off the same or multiple persistent queues. In fact, ElasticCode can auto-scale processors to accommodate the swell of tasks arriving on a queue. In addition, flow processors will be automatically balanced across physical locations to evenly distribute computational load and reduce local resource contention.
9. **Pluggable Backends** - ElasticCode supports various implementations of backend components such as message (e.g. RabbitMQ, SQS) or result storage (SQL, Redis, S3) in addition to allowing you to implement an entire backend (behind the SQL database) yourself.
10. **Real-time Metrics** - ElasticCode processors will support real-time broadcasting of data throughput metrics via subscription web-sockets. This will allow for all kinds of custom integrations and front-end visualizations to see what the network is doing.
11. **Data Analysis** - One of the big goals for ElasticCode is to save important data metrics about the flows and usages so it can be mined by predictive AI models later. This will give your organization key insights into the movement patterns of data.
12. **GIT Integration** - All the code used by processors can be pulled from your own git repositories giving you instant integration into existing devops and CM processes. ElasticCode will let you select which repo and commit version you want a processor to execute code from in your flows.