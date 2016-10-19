Nodes, properties and traversals
================================

Querying graph elements
-----------------------

To get familiar with the graph database, let's have a look at how the source code files are stored in the graph database. Every file is stored in a file node. The following query will give all the file nodes:

.. code-block:: groovy

        g.V().has(NODE_TYPE,TYPE_FILE)


A short explanation of the script. Starting at the graph (``g``), give all vertices (``.V()``) that have (``.has(``) the type (``NODE_TYPE,``) 'file' (``TYPE_FILE)``).

When running the script with the Octopus shell in testing mode, he output will show data structures. On the above input, it will show structures like this:

.. code-block:: json

        {
            "id": 20584,
            "label": "vertex",
            "properties": {
                "_key": [
                    {
                        "id": "dfx-fvs-sl",
                        "value": "83"
                    }
                ],
                "childNum": [
                    {
                        "id": "ff1-fvs-29dx",
                        "value": ""
                    }
                ],
                "code": [
                    {
                        "id": "e8d-fvs-3yd",
                        "value": "/home/tim/RESEARCH/OCTOPUS/timhemel-joern-dev/projects/octopus/data/projects/testdata.tar.gz/src/testdata/test_function.c"
                    }
                ],
                "functionId": [
                    {
                        "id": "f0t-fvs-28lh",
                        "value": ""
                    }
                ],
                "isCFGNode": [
                    {
                        "id": "ft9-fvs-2a6d",
                        "value": ""
                    }
                ],
                "location": [
                    {
                        "id": "eml-fvs-27t1",
                        "value": ""
                    }
                ],
                "type": [
                    {
                        "id": "du5-fvs-1l1",
                        "value": "File"
                    }
                ]
            },
            "type": "vertex"
        }


Element Properties
------------------

Every element in a Gremlin graph can have properties (key/value pairs). An edge can only have a single value for a key, but a vertex can have multiple values for a key. That is why we see lists in the ``properties`` data structure above, for example the value for ``code``. Note that this is a list of dictionaries, because a vertex property can itself also have properties (but this does not work recursively). A more detailed description can be found in the `TinkerPop3 documentation <http://tinkerpop.apache.org/docs/3.0.1-SNAPSHOT/#vertex-properties>`_.

You can ask for an element's properties by adding another step: ``properties()``.

.. code-block:: groovy

        g.V().has(NODE_TYPE,TYPE_FILE).properties()

If we run the above query, we see a list of data structures, but instead of getting one list of properties for each file node, we see 7 separate data structures for each file node:

.. code-block:: json

        {"id": "eml-fvs-27t1", "label": "location", "value": ""},
        {"id": "f0t-fvs-28lh", "label": "functionId", "value": ""},
        {"id": "ff1-fvs-29dx", "label": "childNum", "value": ""},
        {"id": "ft9-fvs-2a6d", "label": "isCFGNode", "value": ""},
        {"id": "dfx-fvs-sl", "label": "_key", "value": "83"},
        {"id": "du5-fvs-1l1", "label": "type", "value": "File"},
        {"id": "e8d-fvs-3yd", "label": "code", "value": "/home/tim/RESEARCH/OCTOPUS/timhemel-joern-dev/projects/octopus/data/projects/testdata.tar.gz/src/testdata/test_function.c"},

You can see a property labeled ``'type'`` and having the value ``'File'``, but in our query we used ``NODE_TYPE`` and ``TYPE_FILE``. These are in fact the same and you can either use the symbolic constants or the actual strings. To see which constants are available, have a look at the file ``languages/joern/_constants.groovy``.

Traversals
----------

We have been talking about queries so far, but in Gremlin, we should speak of so-called traversals. A traversal is an `iterator <https://en.wikipedia.org/wiki/Iterator>`_, that when evaluated gives us values. These can be graph elements, properties, strings or numbers.

In Gremlin, we usually start the traversal with ``g``, which adds the current graph object to the traversal. It is also possible to start an empty traversal with ``__``.

After creating the traversal, you can define steps on it, such as the earlier query:

.. code-block:: groovy

        g.V().has(NODE_TYPE,TYPE_FILE).properties()

The Gremlin documentation distinguishes `five types of traversal steps <http://tinkerpop.apache.org/docs/3.0.1-SNAPSHOT/#graph-traversal-steps>`_:

* ``filter``
* ``map``
* ``flatMap``
* ``sideEffect``
* ``branch``

``filter`` steps take an input and pass it to the output, depending on a certain condition. ``map`` steps take an input and modify it to something different. ``flatMap`` steps do something similar, but one input can result in multiple outputs. ``sideEffect`` steps pass input to output unmodified, but do something else. ``branch`` steps will split off multiple traversals and pass the input to these traversals. In theory, these five basic steps allow us to create any traversal that we would like. In practice a large number of steps is defined because it makes the traversals easier to read and allows traversals to be optimized. It is precisely for the latter reason that `it is discouraged to use these generic steps (lambdas) <http://tinkerpop.apache.org/docs/3.0.1-SNAPSHOT/#a-note-on-lambdas>`_. Which traversal steps are available to you depends on the exact version of TinkerPop3 that is used. Consult the `documentation for the correct version <http://tinkerpop.apache.org/docs/>`_.

According to the above descriptions, the ``properties()`` step is a ``flatMap`` step. The ``has()`` step is a ``filter`` step.

Steps can be chained, to get a more complex iterator. To get all file nodes whose file name ends in ``sample.c``, we can use the following:

.. code-block:: groovy

        g.V().has(NODE_TYPE,TYPE_FILE).has('code',textRegex('.*sample.c$'))

`textRegex <http://titan.thinkaurelius.com/javadoc/1.0.0/com/thinkaurelius/titan/core/attribute/Text.html>`_ is a predicate that is only available in TitanDB. Predicates are used in several steps, such as ``has``, ``is`` and ``where``. More information about predicates can be found in the `Tinkerpop3 documentation <http://tinkerpop.apache.org/docs/3.0.1-SNAPSHOT/#a-note-on-predicates>`_. Using TitanDB specific functions will make your traversal less portable, so use them with care.

If we could only query vertices in the graph and see their properties, we would not be using the full potential of a graph database. The real power of graph databases is in traversing the graph.

Exercises
----------

1. Write a traversal to find all if-statements in the graph. If-statements have the property ``type`` set to ``IfStatement``. `Hint <http://tinkerpop.apache.org/docs/3.0.1-SNAPSHOT/#has-step>`_.

2. Instead of the data structures, output the code string using the `values <http://tinkerpop.apache.org/docs/3.0.1-SNAPSHOT/#vertex-properties>`_ step.

3. What type of step is ``values()``, if you evaluate the following traversal:

        .. code-block:: groovy

                g.V().has('type','Function').values('location','code')



