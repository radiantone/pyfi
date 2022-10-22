
Data Flows
================================
.. toctree::
   :maxdepth: 2

ElasticCode provides a unique and easy way to deploy distributed data flows (sometimes called workflows). These flows are contructed by using the ElasticCode object model and linking them together.

To review the ElasticCode object model, we have the following taxonomy used in an ElasticCode network.

- Nodes
    - Agents
        - Workers
            - Processors
                - Sockets
                    - Tasks
                        -Arguments

For a given processor, multiple sockets can be exposed that allow incoming requests to different functions (tasks) within the processors python module code.
Links between outputs of one socket and inputs of another are established using Plugs.

Each Plug has a source socket and a target socket, such that when the function associated with the source socket completes, its output is used as input to the target socket function.
These requests persist on a queue and execute in an orderly fashion to not stress resources. Since processors are bound to one or more CPUs, they can service requests in parallel but will only execute requests when resources are free to do so.

Because functions are coupled into data flows using loose coupling, you are able to change the topology of your data flow anytime. Execution will follow the path of the current dataflow.

When connecting a Plug to a target Socket, you can specify a specific argument for the target function that the plug is connected to.

For example, consider this target function:

.. code-block:: python

        def add_two(one, two):
            return one+two

*diagram*

It has two arguments `one` and `two` by name. You might have a data flow with two separate inputs to `add_two` where one plug satisfies the `one` argument and the other plug satisfies the `two` argument.
In this design,`add_two` will only trigger once both arguments have `arrived` at the socket. This means arguments can arrive at different times and different orders.