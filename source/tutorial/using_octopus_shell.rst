Using the Octopus Shell
=======================

To interact with the Octopus server, you can use ``octopus-shell``. Sessions on the Octopus server are called *shells*, and they will can be kept alive, so that you can reconnect to the same shell. Re-using the shell saves time, because the standard libraries for the shell do not have to be reloaded all the time.


Connecting to the Octopus Server
--------------------------------

To see which shells are active on the server, simply run ``octopus-shell ls``:

.. code-block:: none

        $ octopus-shell ls
        6000 testCode.tar.gz .python_testCode.tar.gz_0 free

This shows a shell that is connected to the project ``testCode.tar.gz`` running on port 6000, and it is available (``free``).

If there are no available shells, you will need to create one first:

.. code-block:: none

        octopus-shell create testCode.tar.gz

After that, the shell should be in the list:

.. code-block:: none

        6000 testCode.tar.gz noname free

If you paid close attention, you will notice that the shell in the first listing was created by a python script, whereas this shell was created with the ``create`` command.

Once you have a shell available, you can connect to it, and start issuing commands:

.. code-block:: none

        $ octopus-shell co
        Connecting to database 'testCode.tar.gz' on port 6000.
           ___       _
          / _ \  ___| |_ ___  _ __  _   _ ___
         | | | |/ __| __/ _ \| '_ \| | | / __|
         | |_| | (__| || (_) | |_) | |_| \__ \
          \___/ \___|\__\___/| .__/ \__,_|___/
                             |_| Octopus shell

        > 


To exit the shell, you can type control-d (on UNIX or UNIX-like systems).
Typing ``quit`` will exit and delete the shell from the server.

If you have more than one shell, you will need to specify the port number:

.. code-block:: none

        $ octopus-shell co -q 6001

Typing Queries in the Octopus Shell
-----------------------------------


You can type queries (to be more precise: *traversals*) in the Octopus shell and the result will be displayed.

.. code-block:: none

        > g.V().limit(3)
        v[4096]
        v[8192]
        v[12288]

The Octopus shell is good for interactively testing queries. To send queries to the Octopus server from a program is better done with the Python library.


