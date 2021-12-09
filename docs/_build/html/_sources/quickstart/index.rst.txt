
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

Let's look at the sequence of CLI commands needed to build out our flow infrastructure and execute a task. From scratch!
First thing we do below is create a queue. This provides the persistent message broker the definition it needs to allocate a ``message queue`` by the same name for holding task messages.

Next we create a processor, which refers to our gitrepo and defines the module within that codebase we want to expose. It also defines the host where the processor should be run, but that is optional.
We specific a concurrency value of 5 that indicates *the scale* for our processor. This means it will seek to occupy 5 CPUs, allowing it to run in parallel and respond to high-volume message traffic better.

Then we create sockets and attach them to our processor. The socket tells pyfi what specific python function we want to receive messages for and what queue it should use. Lastly, it indicates what processor to be attached to.

Finally, we can run our task and get the result.

.. code-block:: bash

   $ pyfi add queue -n pyfi.queue1 -t direct
   $ pyfi add processor -n proc1 -g https://github.com/radiantone/pyfi-processors -m pyfi.processors.sample -h localhost -c 5
   $ pyfi add socket -n pyfi.processors.sample.do_something -q pyfi.queue1 -pn proc1 -t do_something
   $ pyfi add socket -n pyfi.processors.sample.do_this -q pyfi.queue1 -pn proc1 -t do_this
   $ pyfi task run --socket pyfi.processors.sample.do_this --data "['some data']"
   Do this: ['some data']

Creating Sockets
^^^^^^^^^^^^^^^^
Sockets represent addressable endpoints for python functions hosted by a processor. Remember, the processor points to a gitrepo and defines a python module within that repo.
The socket defines the task (or python function) within the processor python module. Thus, a single processor can have many sockets associated with it. Sockets also declare a queue they will use to pull their requests from.
This allows calls to tasks to be durable and reliable.

The following extract from the above flow defines a socket, gives it a name ``pyfi.processors.sample.do_something``, declares the queue ``pyfi.queue1``, associates it with processor named ``proc1`` and represents the python function/task ``do_something``.

.. code-block:: bash

   $ pyfi add socket -n pyfi.processors.sample.do_something -q pyfi.queue1 -pn proc1 -t do_something

Defining Socket Functions
^^^^^^^^^^^^^^^^^^^^^^^^^

Once you've built out your flow and infrastructure to support it, you can create convenient types that represent your python functions via the Socket class.

For the parallel flow above, we import the .p (or partial) signature from this file, which comes from our Socket we created earlier named ``pyfi.processors.sample.do_something``.
Remember, the socket captures the module (from its parent Processor) and function name within that module you want to run. Think of it like an endpoint with a queue in front of it.

We take one step further in the file below and rename Socket class to Function simply as a linguistic preference in this context.

.. code-block:: python

   from pyfi.client.api import Socket as Function

   do_something = Function(name='pyfi.processors.sample.do_something')
   do_something_p = do_something.p

   do_this = Function(name='pyfi.processors.sample.do_this')
   do_this_p = do_this.p

Once we've created our function definitions above, we can use them like normal python functions as in the parallel workflow below!

Executing Socket Functions
^^^^^^^^^^^^^^^^^^^^^^^^^^

Executing socket functions from python is very easy. Since we can create the socket ahead of time, we only need to refer to it by name as above.

.. code-block:: python

   from pyfi.client.examples.api import do_something_p as do_something

   do_something("Some text!")

The just invoke the function reference as you normally would. If you are using the function within a parallel API structure such as ``parallel``, ``pipeline``, ``funnel`` etc then you should use the ``partial`` (.p, _p) version of the function signature.
This allows PYFI to add arguments to the task when it is invoked. The invocation is deferred so it doesn't happen at the time you declare your workflow. The reason is because your task will execute on thos remote CPU at a time when the workflow reaches that task.
So the .p partial is a ``signature`` for your task in that respect.

Running a Parallel Workflow
---------------------------

.. code-block:: python

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

