
Examples
================================
.. toctree::
   :maxdepth: 2
 

.. code-block:: bash
      :caption: The 'pyfi' command is the single command for building and managing a PYFI network.

      $ pyfi
      Usage: pyfi [OPTIONS] COMMAND [ARGS]...

      Pyfi CLI for managing the pyfi network

      Options:
      --debug         Debug switch
      -d, --db TEXT   Database URI
      --backend TEXT  Task queue backend
      --broker TEXT   Message broker URI
      -i, --ini TEXT  PYFI .ini configuration file
      -c, --config    Configure pyfi
      --help          Show this message and exit.

      Commands:
      add        Add an object to the database
      agent      Run pyfi agent
      api        API server admin
      db         Database operations
      delete     Delete an object from the database
      listen     Listen to a processor output
      ls         List database objects and their relations
      node       Node management operations
      proc       Run or manage processors
      scheduler  Scheduler management commands
      task       Pyfi task management
      update     Update a database object
      web        Web server admin
      whoami     Database login user
      worker     Run pyfi worker

Database
-----------------
.. code-block:: bash
      :caption: PYFI database sub-commands

      $ pyfi db
      Usage: pyfi db [OPTIONS] COMMAND [ARGS]...

      Database operations

      Options:
      --help  Show this message and exit.

      Commands:
      drop     Drop all database tables
      init     Initialize database tables
      json     Dump the database to JSON
      migrate  Perform database migration/upgrade
      rebuild  Drop and rebuild database tables

Objects
-------------------------

There are numerous objects within a PYFI network. Some are infrastructure related, others are service related. Using the PYFI CLI you create, update and manage these objects in the database, which acts as a **single source of truth** for the entire PYFI network.
All the deployed PYFI services (e.g. agents) *react* to changes in the PYFI database. So you could say that PYFI is *reactive* on a distributed, network-scale.

Some of the system objects and CLI commands are shown below.

Queues
------

.. code-block:: bash
      :caption: Add a queue to the database

      $ pyfi add queue --help
      Usage: pyfi add queue [OPTIONS]

      Add queue object to the database

      Options:
      -n, --name TEXT                 [required]
      -t, --type [topic|direct|fanout]
                                       [default: direct; required]
      --help                          Show this message and exit.

Processors
-----------------------

.. code-block:: bash
      :caption: Add a processor to the database

      $ pyfi add processor --help
      Usage: pyfi add processor [OPTIONS]

      Add processor to the database

      Options:
      -n, --name TEXT               Name of this processor  [required]
      -m, --module TEXT             Python module (e.g. some.module.path
                                    [required]

      -h, --hostname TEXT           Target server hostname
      -w, --workers INTEGER         Number of worker tasks
      -r, --retries INTEGER         Number of retries to invoke this processor
      -g, --gitrepo TEXT            Git repo URI  [required]
      -c, --commit TEXT             Git commit id for processor code
      -rs, --requested_status TEXT  The requested status for this processor
      -b, --beat                    Enable the beat scheduler
      -br, --branch TEXT            Git branch to be used for checkouts
      --help                        Show this message and exit.

.. code-block:: bash
      :caption: Specific processor subcommands

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

Calls
-----

.. code-block:: bash
      :caption: Call subcommands

      $ pyfi ls calls --help
      Usage: pyfi ls calls [OPTIONS]

      List queues

      Options:
      -p, --page INTEGER
      -r, --rows INTEGER
      -a, --ascend
      --help              Show this message and exit.


.. code-block:: bash
      :caption: pyfi ls calls
      
      $ pyfi ls calls
      +------+-----+-------------------------------------+--------------------------------------+----------+----------------------------+-------------------------------------+----------------------------+----------------------------+----------+
      | Page | Row |                 Name                |                  ID                  |  Owner   |        Last Updated        |                Socket               |          Started           |          Finished          |  State   |
      +------+-----+-------------------------------------+--------------------------------------+----------+----------------------------+-------------------------------------+----------------------------+----------------------------+----------+
      |  1   |  1  |    pyfi.processors.sample.do_this   | e3f73300-f3fd-4230-ba11-258d4f5a17f4 | postgres | 2021-09-13 19:30:19.933346 |    pyfi.processors.sample.do_this   | 2021-09-13 19:30:19.903573 | 2021-09-13 19:30:19.932491 | finished |
      |  1   |  2  | pyfi.processors.sample.do_something | e3bf09c5-ae45-4772-b301-c394acae3c4e | postgres | 2021-09-13 19:30:19.885993 | pyfi.processors.sample.do_something | 2021-09-13 19:30:19.847282 | 2021-09-13 19:30:19.885440 | finished |
      |  1   |  3  |    pyfi.processors.sample.do_this   | a58de16a-1b92-4acb-81c1-92e81cb6ea56 | postgres | 2021-09-13 19:29:49.944219 |    pyfi.processors.sample.do_this   | 2021-09-13 19:29:49.917225 | 2021-09-13 19:29:49.943415 | finished |
      |  1   |  4  | pyfi.processors.sample.do_something | 58df162a-ac2e-40b7-9e27-635c61a4d9a7 | postgres | 2021-09-13 19:29:49.868975 | pyfi.processors.sample.do_something | 2021-09-13 19:29:49.820097 | 2021-09-13 19:29:49.868109 | finished |
      |  1   |  5  |    pyfi.processors.sample.do_this   | 60d8b91d-1b8b-433c-a289-5704856d37d1 | postgres | 2021-09-13 19:29:19.907705 |    pyfi.processors.sample.do_this   | 2021-09-13 19:29:19.880742 | 2021-09-13 19:29:19.906931 | finished |
      |  1   |  6  | pyfi.processors.sample.do_something | 66c78849-9052-48d0-ae62-59942d544096 | postgres | 2021-09-13 19:29:19.861880 | pyfi.processors.sample.do_something | 2021-09-13 19:29:19.824456 | 2021-09-13 19:29:19.861330 | finished |
      |  1   |  7  |    pyfi.processors.sample.do_this   | e5189a71-9805-492e-a8d7-e5eb2b8d68d3 | postgres | 2021-09-13 19:28:49.873301 |    pyfi.processors.sample.do_this   | 2021-09-13 19:28:49.842724 | 2021-09-13 19:28:49.872176 | finished |
      |  1   |  8  | pyfi.processors.sample.do_something | 35fd3635-743a-4015-acfe-c5a8f62ef65d | postgres | 2021-09-13 19:28:49.812921 | pyfi.processors.sample.do_something | 2021-09-13 19:28:49.789503 | 2021-09-13 19:28:49.812406 | finished |
      |  1   |  9  |    pyfi.processors.sample.do_this   | 4136ebe2-ee96-4b74-ba0e-33d8c5974252 | postgres | 2021-09-13 19:28:19.830508 |    pyfi.processors.sample.do_this   | 2021-09-13 19:28:19.805839 | 2021-09-13 19:28:19.829667 | finished |
      |  1   |  10 | pyfi.processors.sample.do_something | 707f18c5-5708-4c70-81fb-ca0afb30e28b | postgres | 2021-09-13 19:28:19.789542 | pyfi.processors.sample.do_something | 2021-09-13 19:28:19.764792 | 2021-09-13 19:28:19.788999 | finished |
      +------+-----+-------------------------------------+--------------------------------------+----------+----------------------------+-------------------------------------+----------------------------+----------------------------+----------+
      Page 1 of 383 of 3830 total records

.. code-block:: bash
      :caption: pyfi ls call --help

      $ pyfi ls call --help
      Usage: pyfi ls call [OPTIONS]

      List details about a call record

      Options:
      --id TEXT        ID of call
      -n, --name TEXT  Name of call
      -r, --result     Include result of call
      -t, --tree       Show forward call tree
      -g, --graph      Show complete call graph
      -f, --flow       Show all calls in a workflow
      --help           Show this message and exit.

.. code-block:: bash
      :caption: pyfi ls call --id e3bf09c5-ae45-4772-b301-c394acae3c4e
      
      $ pyfi ls call --id e3bf09c5-ae45-4772-b301-c394acae3c4e
      +-------------------------------------+--------------------------------------+----------+----------------------------+-------------------------------------+----------------------------+----------------------------+----------+
      |                 Name                |                  ID                  |  Owner   |        Last Updated        |                Socket               |          Started           |          Finished          |  State   |
      +-------------------------------------+--------------------------------------+----------+----------------------------+-------------------------------------+----------------------------+----------------------------+----------+
      | pyfi.processors.sample.do_something | e3bf09c5-ae45-4772-b301-c394acae3c4e | postgres | 2021-09-13 19:30:19.885993 | pyfi.processors.sample.do_something | 2021-09-13 19:30:19.847282 | 2021-09-13 19:30:19.885440 | finished |
      +-------------------------------------+--------------------------------------+----------+----------------------------+-------------------------------------+----------------------------+----------------------------+----------+
      Provenance
      +--------------------------------------+-------------+-------------+
      |                 Task                 | Task Parent | Flow Parent |
      +--------------------------------------+-------------+-------------+
      | a13ba1e7-78f9-4644-9c29-696dfd89e9e4 |     None    |     None    |
      +--------------------------------------+-------------+-------------+
      Events
      +----------+--------------------------------------+----------+----------------------------+-----------------------------------------------------+
      |   Name   |                  ID                  |  Owner   |        Last Updated        |                         Note                        |
      +----------+--------------------------------------+----------+----------------------------+-----------------------------------------------------+
      | received | 8e8845d5-cd32-40d9-93c7-e95f7500926c | postgres | 2021-09-13 19:30:19.844512 |  Received task pyfi.processors.sample.do_something  |
      |  prerun  | a2507cd1-1d72-4ad1-be74-375aac29f1c4 | postgres | 2021-09-13 19:30:19.874789 | Prerun for task pyfi.processors.sample.do_something |
      | postrun  | f8b5ff03-e0e3-467d-9257-a682f0865581 | postgres | 2021-09-13 19:30:19.886504 |                  Postrun for task                   |
      +----------+--------------------------------------+----------+----------------------------+-----------------------------------------------------+

.. code-block:: bash
      :caption: pyfi ls call --id e3bf09c5-ae45-4772-b301-c394acae3c4e --tree

      $ pyfi ls call --id e3bf09c5-ae45-4772-b301-c394acae3c4e --tree
      pyfi.processors.sample.do_something                   
            └────────────────────┐                        
                     pyfi.processors.sample.do_this    


Listening
---------
The listen command allows you to listen to the pubsub channels associated with queues and processors. A kind of *network sniffer* that displays in real-time the various message traffic, compute results, lifecycle events etc.
You can provide your own custom class to receive the results which is designed to provide a simple and loosely coupled mechanism for system integrations.

.. code-block:: bash
      :caption: Messages will be displayed as they are generated within the network.

      $ pyfi listen --help
      Usage: pyfi listen [OPTIONS]

      Listen to a processor output

      Options:
      -n, --name TEXT     Name of processor  [required]
      -c, --channel TEXT  Listen channel (e.g. task, log, etc)  [required]
      -a, --adaptor TEXT  Adaptor class function (e.g. my.module.class.function)
      --help              Show this message and exit.
      $ pyfi listen --name pyfi.queue1.proc1 --channel task
      Listening to pyfi.queue1.proc1
      {'type': 'psubscribe', 'pattern': None, 'channel': b'pyfi.queue1.proc1.task', 'data': 1}
      {'type': 'pmessage', 'pattern': b'pyfi.queue1.proc1.task', 'channel': b'pyfi.queue1.proc1.task', 'data': b'{"channel": "task", "state": "received", "date": "2021-09-13 19:37:20.094443", "room": "pyfi.queue1.proc1"}'}
      {'type': 'pmessage', 'pattern': b'pyfi.queue1.proc1.task', 'channel': b'pyfi.queue1.proc1.task', 'data': b'{"channel": "task", "state": "running", "date": "2021-09-13 19:37:20.108668", "room": "pyfi.queue1.proc1"}'}
      {'type': 'pmessage', 'pattern': b'pyfi.queue1.proc1.task', 'channel': b'pyfi.queue1.proc1.task', 'data': b'{"module": "pyfi.processors.sample", "date": "2021-09-13 19:37:20.133327", "resultkey": "celery-task-meta-b3feb181-484d-4b98-aba8-daabd07ee3d1", "message": "{\\"module\\": \\"pyfi.processors.sample\\", \\"date\\": \\"2021-09-13 19:37:20.133327\\", \\"resultkey\\": \\"celery-task-meta-b3feb181-484d-4b98-aba8-daabd07ee3d1\\", \\"message\\": \\"\\\\\\"\\\\\\\\\\\\\\"Message:Hello World!\\\\\\\\\\\\\\"\\\\\\"\\", \\"channel\\": \\"task\\", \\"room\\": \\"pyfi.queue1.proc1\\", \\"task\\": \\"do_something\\"}", "channel": "task", "room": "pyfi.queue1.proc1", "task": "do_something", "state": "postrun"}'}

Running an Agent
----------------

.. code-block:: bash
      :caption: PYFI agent subcommand

      $ pyfi agent 
      Usage: pyfi agent [OPTIONS] COMMAND [ARGS]...

      Run pyfi agent

      Options:
      --help  Show this message and exit.

      Commands:
      start  Run pyfi agent server


.. code-block:: bash
      :caption: PYFI agent subcommand

      $ pyfi agent start --help
      Usage: pyfi agent start [OPTIONS]

      Run pyfi agent server

      Options:
      -p, --port INTEGER  Listen port
      --clean             Remove work directories before launch
      -b, --backend TEXT  Message backend URI
      -r, --broker TEXT   Message broker URI
      -c, --config TEXT   Config module.object import (e.g.
                        path.to.module.MyConfigClass

      -q, --queues        Run the queue monitor only
      -u, --user TEXT     Run the worker as user
      -p, --pool INTEGER  Process pool for message dispatches
      -s, --size INTEGER  Maximum number of messages on worker internal queue
      --help              Show this message and exit.

Roles & Users
-------------

.. code-block:: bash
      :caption: PYFI user, role and privilege subcommands

      $ pyfi add user --help
      Usage: pyfi add user [OPTIONS]

      Add user object to the database

      Options:
      -n, --name TEXT      [required]
      -e, --email TEXT     [required]
      -p, --password TEXT  [required]
      --help               Show this message and exit.

      $ pyfi add role --help
      Usage: pyfi add role [OPTIONS]

      Add role object to the database

      Options:
      -n, --name TEXT  [required]
      --help           Show this message and exit.

      $ pyfi add privilege --help
      Usage: pyfi add privilege [OPTIONS]

      Add privilege to the database

      Options:
      -u, --user TEXT  
      -n, --name TEXT  [required]
      -r, --role TEXT
      --help           Show this message and exit.


.. code-block:: bash
      :caption: Creating a user

      $ pyfi add user
      Name: joe
      Email: joe@xyz
      Password: 12345
      CREATE USER joe WITH PASSWORD '12345'
      User "joe" added

.. code-block:: bash
      :caption: Creating a role

      $ pyfi add role -n developer
      bc15ee9d-a208-43a9-82d2-bf0810dc4380:developer:2021-09-15 21:50:40.714192

.. code-block:: bash
      :caption: Adding a privilege to a user

      $ pyfi add privilege -u joe -n ADD_PROCESSOR
      Privilege added     

.. code-block:: bash
      :caption: List a user with role_privileges

      $ pyfi ls user -n joe
      +------+--------------------------------------+----------+---------+
      | Name |                  ID                  |  Owner   |  Email  |
      +------+--------------------------------------+----------+---------+
      | joe  | a8dcf9bb-c821-4d44-82f5-828dceb4cb23 | postgres | joe@xyz |
      +------+--------------------------------------+----------+---------+
      Privileges
      +------+---------------+----------------------------+----------+
      | Name |     Right     |        Last Updated        |    By    |
      +------+---------------+----------------------------+----------+
      | joe  | ADD_PROCESSOR | 2021-09-15 21:46:48.611286 | postgres |
      +------+---------------+----------------------------+----------+

.. code-block:: bash
      :caption: Adding a privilege to a role

      $ pyfi add privilege -r developer -n ADD_PROCESSOR
      Privilege added    

.. code-block:: bash
      :caption: Adding a role to a user
    

Privileges & Rights
---------------------------

A **right** is an atomic string that names a particular **privilege**. It only becomes a privilege when it's associated with a user.
When it's just **a name** we call it a *right*.

.. code-block:: python
      :caption: Available Rights

      rights = ['ALL',
          'CREATE',
          'READ',
          'UPDATE',
          'DELETE',

          'DB_DROP',
          'DB_INIT',

          'START_AGENT',

          'RUN_TASK',
          'CANCEL_TASK',

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