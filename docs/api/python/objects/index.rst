
Objects
================================
.. toctree::
   :maxdepth: 2


.. code-block:: python
      :caption: ElasticCode Python Object API
        import json

        from pyfi.client.api import Plug, Processor, Socket
        from pyfi.client.user import USER
        from pyfi.db.model import AlchemyEncoder

        # Log in a user first
        print("USER", USER)
        # Create a processor
        processor = Processor(
            name="proc1",
            beat=True,
            user=USER,
            module="pyfi.processors.sample",
            branch="main",
            concurrency=6,
            gitrepo="https://user:key@github.com/radiantone/pyfi-processors#egg=pyfi-processor",
        )

        processor2 = Processor(
            name="proc2",
            user=USER,
            module="pyfi.processors.sample",
            hostname="agent1",
            concurrency=6,
            branch="main",
            gitrepo="https://user:key@github.com/radiantone/pyfi-processors#egg=pyfi-processor",
        )

        processor3 = Processor(
            name="proc3",
            user=USER,
            module="pyfi.processors.sample",
            hostname="agent2",
            concurrency=6,
            branch="main",
            gitrepo="https://user:pword@github.com/radiantone/pyfi-processors#egg=pyfi-processor",
        )

        # Create a socket on the processor to receive requests for the do_something python function(task)
        do_something = Socket(
            name="pyfi.processors.sample.do_something",
            user=USER,
            interval=5,
            processor=processor,
            queue={"name": "pyfi.queue1"},
            task="do_something",
        )

        print(json.dumps(do_something.socket, indent=4, cls=AlchemyEncoder))
        # Create a socket on the processor to receive requests for the do_this python function(task)
        do_this = Socket(
            name="pyfi.processors.sample.do_this",
            user=USER,
            processor=processor2,
            queue={"name": "pyfi.queue2"},
            task="do_this",
        )
        do_this2 = Socket(
            name="pyfi.processors.sample.do_this",
            user=USER,
            processor=processor3,
            queue={"name": "pyfi.queue3"},
            task="do_this",
        )

        do_something2 = Socket(
            name="proc2.do_something",
            user=USER,
            processor=processor2,
            queue={"name": "pyfi.queue1"},
            task="do_something",
        )

        # Create a plug that connects one processor to a socket of another
        plug = Plug(
            name="plug1",
            processor=processor,
            user=USER,
            source=do_something,
            queue={"name": "pyfi.queue3"},
            target=do_this,
        )

