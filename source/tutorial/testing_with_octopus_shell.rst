Testing with octopus-shell
==========================

If you want to test queries, this is best done in an environment where results
are reproducible. This means: having the same initial state before sending the
query. For that, we will need to create a fresh Octopus shell and exit it again
afterwards.

For this, ``octopus-shell connect`` offers two extra options:

``-n`` will always start a fresh shell, and ``-x`` will destroy it afterwards.

The results will be a string representation of the actual results. If the ``-r`` option is given to the ``connect`` command, it will show the actual data structures in JSON format (one JSON structure per result). This turns out to be very useful when exploring the code database and when debugging queries.

Testing is not just debugging however. With joern comes a framework for automated testing of queries, called ``gremtest``. It is similar to most unit test frameworks and helps you to not go insane when a Gremlin query does not work the way you expected. We will talk more about testing and debugging in section XXX.

If you want to try the queries in this tutorial, save the query into a file, for example ``testquery.groovy``. Then you can run the following command:

.. code-block:: bash

        $ octopus-shell co -d testdata.tar.gz -rnx testquery.groovy 

In this example, ``testdata.tar.gz`` is the name of the code database.


