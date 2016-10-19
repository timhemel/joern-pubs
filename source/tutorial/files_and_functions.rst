Files and functions
===================

If you have tried the exercise from :doc:`vertex_steps_and_edges`, you may have discovered the edge type ``IS_FILE_OF``, which runs from a file node to a function node. A function is the basic unit in Joern.

joern-list-files and joern-list-funcs
-------------------------------------

The tool ``joern-list-files`` will list all files in the project that match a certain pattern. It will output the ``id`` of the file node and the file name.

``joern-list-funcs`` will list all functions in the project whose name matches a certain pattern. It will output the ``id`` of the function node, the name of the function, and the file name.

Exercises
---------

1. In ``joern-list-funcs``, a query like the following is used:

.. code-block:: groovy

        g.V().has('type', 'Function')
        .has('code', textRegex('.*sample$'))
        .as('func')
        .functionToFile().as('file').select('func', 'file')

Read the `documentation on the select step <http://tinkerpop.apache.org/docs/3.0.1-SNAPSHOT/#select-step>`_ and explain how this query works.


