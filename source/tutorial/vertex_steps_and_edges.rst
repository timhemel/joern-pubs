Vertex Steps and Edges
======================

We have seen what traversals are and the different type of steps that exist. We have seen how to query vertices in the graph and how to see which properties they have. To get more useful information out of the graph, we will need to traverse the graph.

Edges and their properties
--------------------------

Vertices in a graph are connected by edges. In Gremlin, all edges have a direction, identified by a source and a destination. Furthermore, edges have a label and can have properties, but one property can only have one value and there is no such thing as meta properties. A typical edge data structure looks like this:

.. code-block:: json

        {
            "id": "oit4w-m4g-2exh-6f4",
            "inV": 8320,
            "inVLabel": "vertex",
            "label": "REACHES",
            "outV": 28672,
            "outVLabel": "vertex",
            "properties": {
                "var": "a"
            },
            "type": "edge"
        }

Just as in a vertex, we see the ``properties`` dictionary (note that it is indeed simpler), the ``label``, the outgoing vertex ``outV`` (the source vertex), the incoming vertex ``inV`` (the destination vertex) and the identifier ``id``.

The label is not a property, and searching for edges with a certain label cannot be done with the ``has`` filter step. You will need to use ``hasLabel`` instead.
 
Stepping from, to, and over the edge
------------------------------------

We can ask for edges using the ``E()`` step, similar to the ``V()`` step for vertices. From a vertex, we can ask for outgoing edges using ``outE``, incoming edges using ``inE`` or all node edges using ``bothE``. These steps can take a label as its argument, so ``inE('REACHES')`` will give the incoming edges with label ``REACHES``.

Similarly, from an edge we can get back to a vertex with ``outV()`` (outgoing vertex), ``inV`` (incoming vertex), and ``bothV`` (any vertex). There is also ``otherV`` which will move to the vertex other than the one that was moved from.

So, to move from one node to the other, we can use ``outE`` to get the outgoing edge and use ``inV`` to get its incoming vertex:

.. code-block:: groovy

	g.V(123).outE().inV()


Because this is a bit cumbersome, these two steps are combined into one:

.. code-block:: groovy

	g.V(123).out()


Similarly, there is ``in`` and ``both``. These steps also take a label as its argument. A more thorough explanation can be found in the `Tinkerpop3 documentation <http://tinkerpop.apache.org/docs/3.0.1-SNAPSHOT/#vertex-steps>`_.

Armed with this knowledge, let's explore the Joern graph.

Exercises
----------

1. Load the code for this tutorial. Get the file node for ``vertex_steps_and_edges.c`` and output the names of all function that are defined in that file. Hint: from the file node, get a list of all edges and see which one would be appropriate to follow.

