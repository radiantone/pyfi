
Lambda
================================
.. toctree::
   :maxdepth: 2


.. code-block:: python
      :caption: ElasticCode Python Lambda API

        from pyfi.client.api import funnel, parallel, pipeline
        from pyfi.client.example.api import do_something_p as do_something
        from pyfi.client.example.api import do_this_p as do_this

        """
        An example app on top of pyfi. References existing infrastructure and then runs complex workflows and parallel operations on it
        """
        _pipeline = pipeline(
            [
                do_something("One"),
                do_something("Two"),
                parallel(
                    [
                        do_this("Four"),
                        do_this("Five"),
                    ]
                ),
                do_this("Three"),
            ]
        )
        print(_pipeline().get())
        _parallel = parallel([_pipeline, do_something("Six"), do_something("Seven")])

        _funnel = funnel(
            [do_something("Eight"), _parallel, do_something("Nine")], do_something("A")
        )

        _funnel2 = funnel([_parallel, do_something("Ten")], do_something("B"))

        _funnel3 = funnel([_funnel, _funnel2])

        result = _funnel3(do_something("Eleven"))
        print("FUNNEL: ", result.get())
