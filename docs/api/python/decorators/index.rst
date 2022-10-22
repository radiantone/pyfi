
Decorators
================================
.. toctree::
   :maxdepth: 2


.. code-block:: python
      :caption: ElasticCode Python decorator API
        """
        Decorator API for Flow. Defines network from plain old classes and methods.
        """
        import os

        from pyfi.client.api import ProcessorBase
        from pyfi.client.decorators import plug, processor, socket


        @processor(
            name="proc2",
            gitrepo=os.environ["GIT_REPO"],
            module="pyfi.processors.sample",
            concurrency=1,
        )
        class ProcessorB(ProcessorBase):
            """Description"""

            @socket(
                name="proc2.do_this",
                processor="proc2",
                arguments=True,
                queue={"name": "sockq2"},
            )
            def do_this(message):
                from random import randrange

                print("Do this!", message)
                message = "Do this String: " + str(message)
                graph = {
                    "tag": {"name": "tagname", "value": "tagvalue"},
                    "name": "distance",
                    "value": randrange(50),
                }
                return {"message": message, "graph": graph}


        @processor(
            name="proc1",
            gitrepo=os.environ["GIT_REPO"],
            module="pyfi.processors.sample",
            concurrency=7,
        )
        class ProcessorA(ProcessorBase):
            """Description"""

            def get_message(self):
                return "Self message!"

            @plug(
                name="plug1",
                target="proc2.do_this",  # Must be defined above already (prevents cycles)
                queue={
                    "name": "queue1",
                    "message_ttl": 300000,
                    "durable": True,
                    "expires": 200,
                },
            )
            @socket(
                name="proc1.do_something",
                processor="proc1",
                beat=False,
                interval=15,
                queue={"name": "sockq1"},
            )
            def do_something(message):
                """do_something"""
                from random import randrange

                message = "TEXT:" + str(message)
                graph = {
                    "tag": {"name": "tagname", "value": "tagvalue"},
                    "name": "temperature",
                    "value": randrange(10),
                }
                return {"message": message, "graph": graph}


        @processor(
            name="proc3",
            gitrepo=os.environ["GIT_REPO"],
            module="pyfi.processors.sample",
            concurrency=1,
        )
        class ProcessorC(ProcessorBase):
            """Description"""

            def get_message(self):
                return "Self message!"

            @plug(
                name="plug2",
                target="proc2.do_this",  # Must be defined above already (prevents cycles)
                queue={
                    "name": "queue2",
                    "message_ttl": 300000,
                    "durable": True,
                    "expires": 200,
                },
            )
            @socket(
                name="proc3.do_something",
                processor="proc3",
                beat=False,
                interval=5,
                queue={"name": "sockq3"},
            )
            def do_something(message):
                """do_something"""
                from random import randrange

                message = "TEXT2:" + str(message)
                graph = {
                    "tag": {"name": "tagname", "value": "tagvalue"},
                    "name": "temperature",
                    "value": randrange(10),
                }
                return {"message": message, "graph": graph}


        if __name__ == "__main__":
            print("Network created.")



