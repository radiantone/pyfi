![logo](./screens/pyfi.svg)

A distributed data flow and computation system that runs on transactional messaging infrastructure.

## Introduction
PYFI differs from other dataflow engines in that it is fully distributed and runs "at-scale" across heterogeneous infrastructure and computational resources.

It establishes a logical directed-acyclic-graph (DAG) overlay network across compute nodes and executes your custom processor scripts (python, node, bash).

Using the power of reliable, transactional messaging, compute tasks are never lost, discarded or undone. Fault tolerance and load-balancing are intrinsic qualities of PYFI and not tacked on as a separate process, which itself would be a failure point.

## High-Level Architecture
![architecture1](./screens/architecture1.png)

## Design Goals

As the name suggests, PYFI is a spiritual offshoot of Apache NIFI, which is a visual dataflow engine that runs inside a Java virtual machine.
However, PYFI is designed to be more broad in terms of design and scope which we will discuss below.

Some important design goals for this technology are:

1. **Fault-Tolerant** - PYFI runs as a distributed network of logical compute processors that have redundancy and load-balancing built in.
2. **At-Scale** - This phrase is important. It means that the logical constructs (e.g. pyfi processors) run at the scale of the hardware (e.g. CPU processors), or physical constructs with impedence.
3. **Secure** - All the functional components in PYFI (database, broker, storage, cache) have security built in.
4. **Dynamic** - The topology and behavior of a PYFI network can be adjusted and administered in real-time without taking down the entire network. Because PYFI is not a single VM controlling everything, you can add/remove update components without negatively impacting the functionality of the system.
5. **Distributed** - As was mentioned above, everything in PYFI is inherently distributed, down to the processors. There is no physical centralization of any kind. 
6. **Performance** - PYFI is built on mature technology stack that is capable of high-throughput message traffic.
7. **Reliability** - The distributed queue paradigm used by PYFI allows for every processor in your dataflow to consume and acknowledge message traffic from its inbound queues and write to outbound queues. These durable queues persist while processors consume messages off them.
8. **Scalability** - Processors can scale across CPUs, Machines and networks, consuming message traffic off the same or multiple persistent queues. In fact, PYFI can auto-scale processors to accommodate the swell of tasks arriving on a queue. In addition, pyfi processors will be automatically balanced across physical locations to evenly distribute computational load and reduce local resource contention.
9. **Pluggable Backends** - PYFI supports various implementations of backend components such as message (e.g. RabbitMQ, SQS) or result storage (SQL, Redis, S3) in addition to allowing you to implement an entire backend (behind the SQL database) yourself.
10. **Real-time Metrics** - PYFI processors will support real-time broadcasting of data throughput metrics via subscription web-sockets. This will allow for all kinds of custom integrations and front-end visualizations to see what the network is doing.
11. **Data Analysis** - One of the big goals for PYFI is to save important data metrics about the flows and usages so it can be mined by perdictive AI models later. This will give your organization key insights into the movement patterns of data.
12. **GIT Integration** - All the code used by processors can be pulled from your own git repositories giving you instant integration into existing devops and CM processes. PYFI will let you select which repo and commit version you want a processor to execute code from in your flows.

## Detailed Architecture

### Why a SQL Database?
Some of you might be asking why a SQL database is the center abstraction point of PYFI, SQL databases have been around for decades! Let me explain.

There are some important enterprise qualities we want from the logical database that governs the structure and behavior of a PYFI network.

* ***Constraints*** - PYFI data models should adhere to logical constraints that maintain the integrity of the network design. This prevents any errors in the data model that might cause the network to not perform.
* ***Transactions*** - Similar to the nature of the message/task layer, we want to provide transactional semantics to the data layer so sets of logical changes can be applied in an atomic fashion - enforcing constraints as well.
* ***Security*** - Row level security is built into the database and allows us to control who is able to see what without having to implement a weaker form of this at the application layer
* ***Scaling*** - SQL databases such as Postgres have mature scaling mechanics that allow them to cluster and scale appropriately.
* ***Administration*** - Mature tools exist to administer and manage SQL databases that don't need to be reinvented

Coupling the PYFI physical network from the logical model through a transactional databases allows for future implementation-independence of a particular PYFI network.
All the existing PYFI CLI tools that operate on the database will continue to work as is, if you choose to implement a different backend.

### Advanced UI

PYFI uses a custom built, modern User Interface derived from the core design of NIFI but extended in meaningful ways. You can preview the PYFI UI in the [pyfi-ui](#https://github.com/radiantone/pyfi-ui) repository.

**Real Time Streaming Analytics**
![screen1](./screens/pyfi7.png)

**Real Time Coding**
![screen1](./screens/pyfi8.png)

**Advanced Workflows with Embedded Subflows**
![screen1](./screens/screen16.png)

## Outline

*Under Construction*

* [Overview](#overview)
* [Design Goals](#design-goals)  
* [Architecture](#architecture)
  * [Models](#models)
    * [Nodes](#nodes)
    * [Agents](#agents)
    * [Processors](#processors)
    * [Workers](#workers)
    * [Tasks](#tasks)
* [CLI](#cli)  
* [Building Dataflows](#building-dataflows)

