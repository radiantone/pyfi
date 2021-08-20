![logo](./screens/pyfi.svg)

A distributed data flow and computation system that runs on transactional messaging infrastructure. It is designed to operate as a NVM Networked-Virtual-Machine by implementing distributed logic over hardware CPU/GPU processors and is suitable for all kinds of computational tasks.

*NOTE: This repo will initially contain the core design documentation for PYFI but eventually I will commit the code here. Currently looking for sponsors to support the effort. If curious or interested, please reach out to me at darren@ontrenet.com.*
## Introduction
PYFI differs from other dataflow engines in that it is fully distributed and runs "at-scale" across heterogeneous infrastructure and computational resources.

It establishes a logical directed-acyclic-graph (DAG) overlay network across compute nodes and executes your custom processor scripts (python, node, bash).

Using the power of reliable, transactional messaging, compute tasks are never lost, discarded or undone. Fault tolerance and load-balancing are intrinsic qualities of PYFI and not tacked on as a separate process, which itself would be a failure point.

## Outline

*Under Construction*

* [Overview](#overview)
* [High Level Architecture](#high-level-architecture)
* [Tech Stack](#tech-stack)
* [Design Goals](#design-goals)  
* [Detailed Architecture](#detailed-architecture)
  * [Why A SQL Database?](#why-a-sql-database)
  * [Data Model](#data-model)
  * [Security Model](#security-model)
* [Command Line Interface](#command-line-interface)  
* [System Objects](#system-objects)
  * [Nodes](#nodes)
  * [Agents](#agents)
  * [Processors](#processors)
  * [Workers](#workers)
  * [Tasks](#tasks)
* [Building Dataflows](#building-dataflows)
* [Stack Tools](#stack-tools)

## High-Level Architecture
The following diagram shows one cross-section of the current *reference implementation* of PYFI. Since everything behind the database can be implemented in a variety of ways, this architecture is not absolute.

![architecture1](./screens/architecture1.png)

## Tech Stack
The following diagram shows the technology stack for the reference implementation. It uses entirely FOSS software that is mature, open and in most cases supported by a commercial entity.
All of these components provide instant, out-of-the-box functionality that contributes to the PYFI system ensemble and have proven their usefulness in enterprise production settings.

*NOTE: You are not forced to use any of these tools and can use other compatible tools, make your own, or replace the backend component entirely*

![techstack](./screens/techstack.png)
## Design Goals

As the name suggests, PYFI is a spiritual offshoot of [Apache NIFI](https://nifi.apache.org/) except built using a python stack for running python (and other scripting languages) processors.
However, PYFI is designed to be more broad in terms of design and scope which we will discuss below.

Some important design goals for this technology are:

1. **Fault-Tolerant** - PYFI runs as a distributed network of logical compute processors that have redundancy and load-balancing built in.
2. **At-Scale** - This phrase is important. It indicates that the logical constructs (e.g. pyfi processors) run at the scale of the hardware (e.g. CPU processors), meaning there is a 1-1 correlation (physical mapping) between hardware processors and pyfi processors.
3. **Secure** - All the functional components in PYFI (database, broker, storage, cache) have security built in.
4. **Dynamic** - The topology and behavior of a PYFI network can be adjusted and administered in real-time without taking down the entire network. Because PYFI is not a single VM controlling everything, you can add/remove update components without negatively impacting the functionality of the system.
5. **Distributed** - As was mentioned above, everything in PYFI is inherently distributed, down to the processors. There is no physical centralization of any kind. 
6. **Performance** - PYFI is built on mature technology stack that is capable of high-throughput message traffic.
7. **Reliability** - The distributed queue paradigm used by PYFI allows for every processor in your dataflow to consume and acknowledge message traffic from its inbound queues and write to outbound queues. These durable queues persist while processors consume messages off them.
8. **Scalability** - Processors can scale across CPUs, Machines and networks, consuming message traffic off the same or multiple persistent queues. In fact, PYFI can auto-scale processors to accommodate the swell of tasks arriving on a queue. In addition, pyfi processors will be automatically balanced across physical locations to evenly distribute computational load and reduce local resource contention.
9. **Pluggable Backends** - PYFI supports various implementations of backend components such as message (e.g. RabbitMQ, SQS) or result storage (SQL, Redis, S3) in addition to allowing you to implement an entire backend (behind the SQL database) yourself.
10. **Real-time Metrics** - PYFI processors will support real-time broadcasting of data throughput metrics via subscription web-sockets. This will allow for all kinds of custom integrations and front-end visualizations to see what the network is doing.
11. **Data Analysis** - One of the big goals for PYFI is to save important data metrics about the flows and usages so it can be mined by predictive AI models later. This will give your organization key insights into the movement patterns of data.
12. **GIT Integration** - All the code used by processors can be pulled from your own git repositories giving you instant integration into existing devops and CM processes. PYFI will let you select which repo and commit version you want a processor to execute code from in your flows.

## Detailed Architecture

### Why a SQL Database?
Some of you might be asking why a SQL database is the center abstraction point of PYFI, SQL databases have been around for decades! Let me explain.

There are some important enterprise qualities we want from the logical database that governs the structure and behavior of a PYFI network.

* ***Constraints*** - PYFI data models should adhere to logical constraints that maintain the integrity of the network design. This prevents any errors in the data model that might cause the network to not perform. It also protects against any errors introduced by humans.
* ***Transactions*** - Similar to the nature of the message/task layer, we want to provide transactional semantics to the data layer so sets of logical changes can be applied in an atomic fashion. This ensures your network is not caught in an inconsistent (or partial) state when making design changes.
* ***Security*** - Row level security is built into the database and allows us to control who is able to see what without having to implement a weaker form of this at the application layer. By design, the pyfi stack captures access control semantics ***all the way down to the data**.* 
* ***Scaling*** - SQL databases such as Postgres have mature scaling mechanics that allow them to cluster and scale appropriately.
* ***Administration*** - Mature tools exist to administer and manage SQL databases that don't need to be reinvented.

Coupling the PYFI physical network from the logical model through a transactional database allows for future implementation-independence of a particular PYFI network.
All the existing PYFI CLI tools that operate on the database will continue to work as is, if you choose to implement a different backend.

### Data Model

The data model is the system abstraction behind which the PYFI reference implementation operates. Services monitor the data models and reflect the semantics of the data in the PYFI network state.

For example, if a processor model *requested_status* is changed to "STOPPED" then the agent responsible for that processor will stop the processor and put its *status* field to "STOPPED".

Simply put, the PYFI network "reacts" to the current state of the database.

![datamodel](./screens/pyfi-data-model.png)

### Security Model

PYFI uses a fine grained access control model for user actions against the data model (via UI or CLI or API). At the database level this is also enforced with RLS (Row Level Security) features of Postgres (or your database of choice).
It is vital to the security model of PYFI to implement access control all the way through the stack down to the row data.

Using the CLI you can add and remove privileges for individual users and view their current privileges.

```python
$ pyfi add privilege --user darren --name DELETE

# You can only add privileges that are named in the list of rights further below. Trying to add something not in this list will result in an error.

$ pyfi ls user --name darren
+--------+--------------------------------------+----------+------------+
|  Name  |                  ID                  |  Owner   |   Email    |
+--------+--------------------------------------+----------+------------+
| darren | a725b5ff-bb60-401a-a79a-7bfcb87dfc93 | postgres | d@some.com |
+--------+--------------------------------------+----------+------------+
Privileges
+--------+--------+----------------------------+----------+
|  Name  | Right  |        Last Updated        |    By    |
+--------+--------+----------------------------+----------+
| darren | CREATE | 2021-08-18 08:59:42.749164 | postgres |
| darren | DELETE | 2021-08-19 08:36:29.922190 | postgres |
+--------+--------+----------------------------+----------+
```

Here is an initial list of the privileges ("rights") that can be assigned to a user.

```python

rights = [  'ALL',
            'CREATE',
            'READ',
            'UPDATE',
            'DELETE',

            'DB_DROP',
            'DB_INIT',

            'START_AGENT',

            'RUN_TASK',
            'CANCEL_TASK',

            'UPDATE_PROCESSOR',
            'DELETE_PROCESSOR',
            'START_PROCESSOR',
            'STOP_PROCESSOR',
            'PAUSE_PROCESSOR',
            'RESUME_PROCESSOR',
            'LOCK_PROCESSOR',
            'UNLOCK_PROCESSOR',
            'VIEW_PROCESSOR',
            'VIEW_PROCESSOR_CONFIG',
            'VIEW_PROCESSOR_CODE',
            'EDIT_PROCESSOR_CONFIG',
            'EDIT_PROCESSOR_CODE'

            'LS_PROCESSORS',
            'LS_USERS',
            'LS_USER',
            'LS_PLUGS',
            'LS_SOCKETS',
            'LS_QUEUES',
            'LS_AGENTS',
            'LS_NODES',
            'LS_SCHEDULERS',
            'LS_WORKERS',

            'ADD_PROCESSOR',
            'ADD_AGENT',
            'ADD_NODE',
            'ADD_PLUG',
            'ADD_PRIVILEGE',
            'ADD_QUEUE',
            'ADD_ROLE',
            'ADD_SCHEDULER',
            'ADD_SOCKET',
            'ADD_USER',

            'UPDATE_PROCESSOR',
            'UPDATE_AGENT',
            'UPDATE_NODE',
            'UPDATE_PLUG',
            'UPDATE_PRIVILEGE',
            'UPDATE_QUEUE',
            'UPDATE_ROLE',
            'UPDATE_SCHEDULER',
            'UPDATE_SOCKET',
            'UPDATE_USER',

            'DELETE_PROCESSOR',
            'DELETE_AGENT',
            'DELETE_NODE',
            'DELETE_PLUG',
            'DELETE_PRIVILEGE',
            'DELETE_QUEUE',
            'DELETE_ROLE',
            'DELETE_SCHEDULER',
            'DELETE_SOCKET',
            'DELETE_USER',

            'READ_PROCESSOR',
            'READ_AGENT',
            'READ_NODE',
            'READ_PLUG',
            'READ_PRIVILEGE',
            'READ_QUEUE',
            'READ_ROLE',
            'READ_SCHEDULER',
            'READ_SOCKET',
            'READ_USER'
            ]
```
## Command Line Interface

One of the design goals for PYFI was to allow both Graphical and Command line User Interfaces. A CLI will open up access to various server-side automations, devops pipelines and human sysops that can interact with the PYFI network through a remote console.

All constructs within PYFI can be created, deleted, updated or otherwise managed via the CLI as well as the GUI, again, adhering to the principle that architecture is logically designed and loosely coupled in ways that enable more freedom and technology independence if desired.

Here is a sample script that builds a distributed flow using just the CLI

```bash

pyfi add node -n node1 -h phoenix
pyfi add node -n node2 -h radiant 
pyfi add node -n node3 -h miko
pyfi add scheduler --name sched1

pyfi scheduler -n sched1  add --node node1
pyfi scheduler -n sched1  add --node node2
pyfi scheduler -n sched1  add --node node3

pyfi add processor -n proc1 -g https://github.com/radiantone/pyfi-processors -m pyfi.processors.sample -t do_something
pyfi add queue -n pyfi.queue1 -t direct
pyfi add queue -n pyfi.queue2 -t direct
pyfi add queue -n pyfi.queue3 -t direct

pyfi add outlet -n proc1.outlet1 -q pyfi.queue1 -pn proc1
pyfi add plug -n plug1 -q pyfi.queue2 -pn proc1
pyfi add plug -n plug3 -q pyfi.queue3 -pn proc1
pyfi add processor -n proc2 -g https://github.com/radiantone/pyfi-processors -m pyfi.processors.sample -t do_this -h radiant
pyfi add outlet -n proc2.outlet1 -q pyfi.queue2 -pn proc2

pyfi add processor -n proc4 -g https://github.com/radiantone/pyfi-processors -m pyfi.processors.sample -t do_something -h radiant
pyfi add outlet -n proc4.outlet1 -q pyfi.queue1 -pn proc4

pyfi add processor -n proc3 -g https://github.com/radiantone/pyfi-processors -m pyfi.processors.sample -t do_this -h miko
pyfi add outlet -n proc3.outlet1 -q pyfi.queue3 -pn proc3

```

Here are some sample help screens from the CLI.

**Top level pyfi help screen**

```bash
$ pyfi
Usage: pyfi [OPTIONS] COMMAND [ARGS]...

  Pyfi CLI for managing the pyfi network

Options:
  --debug         Debug switch
  -d, --db TEXT   Database URI
  -i, --ini TEXT  PYFI .ini configuration file
  -c, --config    Configure pyfi
  --help          Show this message and exit.

Commands:
  add        Add an object to the database
  agent      Run pyfi agent
  api        API server admin
  db         Database operations
  delete     Delete an object from the database
  ls         List database objects
  node       Node management operations
  proc       Run or manage processors
  scheduler  Scheduler management commands
  task       Pyfi task management
  update     Update a database object
  web        Web server admin
```
**Adding various objects to the PYFI network database**
```bash
$ pyfi add
Usage: pyfi add [OPTIONS] COMMAND [ARGS]...

  Add an object to the database

Options:
  --id TEXT  ID of object being added
  --help     Show this message and exit.

Commands:
  agent      Add agent object to the database
  node       Add node object to the database
  outlet     Add outlet to a processor
  plug       Add plug to a processor
  processor  Add processor to the database
  queue      Add queue object to the database
  role       Add role object to the database
  scheduler  Add scheduler object to the database
  user       Add user object to the database
```

**Running & managing distributed processors**
```bash
$ pyfi proc
Usage: pyfi proc [OPTIONS] COMMAND [ARGS]...

  Run or manage processors

Options:
  --id TEXT  ID of processor
  --help     Show this message and exit.

Commands:
  pause    Pause a processor
  remove   Remove a processor
  restart  Start a processor
  resume   Pause a processor
  start    Start a processor
  stop     Stop a processor
```

**Listing objects in the database**
```bash
$ pyfi ls
Usage: pyfi ls [OPTIONS] COMMAND [ARGS]...

  List database objects

Options:
  --help  Show this message and exit.

Commands:
  agents      List agents
  nodes       List queues
  outlets     List outlets
  plugs       List agents
  processors  List processors
  queues      List queues
  schedulers  List queues
  users       List users
  workers     List workers
```
### Advanced UI

PYFI uses a custom built, modern User Interface derived from the core design of NIFI but extended in meaningful ways. You can preview the PYFI UI in the [pyfi-ui](https://github.com/radiantone/pyfi-ui) repository.

**Real Time Streaming Analytics**
![screen1](./screens/pyfi7.png)

**Real Time Coding**
![screen1](./screens/pyfi8.png)

**Advanced Workflows with Embedded Subflows**
![screen1](./screens/screen16.png)

## Stack Tools

The follow section shows screenshots of the tech stack UI tools.

### pgAdmin

pgadmin is the UI for postgres.

![pgadmin](./screens/pgadmin.png)

### Portainer

Manage your docker container stack

![portainer](./screens/portainer.png)

### Redis Insights

Manage your cache and task results datastore

![portainer](./screens/redis.png)

### RabbitMQ Admin UI

Manage your message broker and queues

![rabbitmq](./screens/rabbitmq.png)

### Flower

Manage your task queues

![flower](./screens/flower.png)

### Kibana

Build dashboards from your logs and long-term persistence

![kibana](./screens/kibana.png)

### Amplify

Monitor your network reverse proxy (NGINX)

![amplify](./screens/amplify.png)