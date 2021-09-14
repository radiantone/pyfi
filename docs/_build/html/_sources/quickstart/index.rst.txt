
Quickstart
================================
.. toctree::
   :maxdepth: 2


Bringing up the Stack
---------------------

.. code-block:: bash

   $ docker-compose up

Configuring PYFI CLI
--------------------

.. code-block:: bash

   $ pyfi --config
   Database connection URI [postgresql://postgres:pyfi101@phoenix:5432/pyfi]: 
   Result backend URI [redis://localhost]: 
   Message broker URI [pyamqp://localhost]: 
   Configuration file created at /home/user/pyfi.ini

Initialize the Database
-----------------------

.. code-block:: bash

   $ pyfi db init
   Enabling security on table action
   Enabling security on table event
   Enabling security on table flow
   Enabling security on table jobs
   Enabling security on table log
   Enabling security on table privilege
   Enabling security on table queue
   Enabling security on table queuelog
   Enabling security on table role
   Enabling security on table scheduler
   Enabling security on table settings
   Enabling security on table task
   Enabling security on table user
   Enabling security on table node
   Enabling security on table processor
   Enabling security on table role_privileges
   Enabling security on table user_privileges
   Enabling security on table user_roles
   Enabling security on table agent
   Enabling security on table plug
   Enabling security on table socket
   Enabling security on table call
   Enabling security on table plugs_queues
   Enabling security on table plugs_source_sockets
   Enabling security on table plugs_target_sockets
   Enabling security on table sockets_queues
   Enabling security on table worker
   Enabling security on table calls_events
   Database create all schemas done.

The PYFI CLI
------------

.. code-block:: bash

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
   
Creating Your First Flow
------------------------

.. code-block:: bash

   $ pyfi add queue -n pyfi.queue1 -t direct
   $ pyfi add processor -n proc1 -g https://github.com/radiantone/pyfi-processors -m pyfi.processors.sample -h localhost
   $ pyfi add socket -n proc1.socket1 -q pyfi.queue1 -pn proc1 -t do_something
   $ pyfi add socket -n proc1.socket2 -q pyfi.queue1 -pn proc1 -t do_this
   $ pyfi task run --socket proc1.socket2 --data "['some data']"
   Do this: ['some data']

Running a Parallel Workflow
---------------------------

.. code-block:: bash

   from pyfi.client.api import parallel, pipeline, funnel
   from pyfi.client.example.api import do_something_p as do_something

   # Create a pipeline that executes tasks sequentially, passing result to next task
   _pipeline = pipeline([
      do_something("One"),
      do_something("Two"),
      # Create a parallel structure that executes tasks in parallel and returns the 
      # result list
      parallel([
         do_something("Four"),
         do_something("Five"),
      ]),
      do_something("Three")])

   # Create another parallel structure using the above pipeline as one of its tasks
   _parallel = parallel([
      _pipeline,
      do_something("Six"),
      do_something("Seven")])

   # Create a funnel structure that executes all its tasks passing the result to the
   # single, final task
   _funnel = funnel([
      do_something("Eight"),
      _parallel,
      do_something("Nine")])

   # Gather the result from the _funnel and send it to do_something("Four")
   print("FUNNEL: ", _funnel(do_something("Four")).get())
