Using Python
============

To connect to the Octopus server from python, you can use the ``octopus.server.DBInterface`` module:

.. code-block:: python

        #!/usr/bin/env python3

        from octopus.server.DBInterface import DBInterface

        projectName = 'testCode.tar.gz'
        query = """g.V().limit(3)"""

        db = DBInterface()
        db.connectToDatabase(projectName)

        result = db.runGremlinQuery(query)
        for x in result:
            print(x)

The result is printed as a Python data structure.

.. code-block:: python

        {'properties': {'_key': [{'id': 'oegow-35s-sl', 'value': '11'}], 'functionId': [{'id': 'oei9s-35s-28lh', 'value': '3'}], 'type': [{'id': 'oeh34-35s-1l1', 'value': 'Identifier'}], 'isCFGNode': [{'id': 'oej28-35s-2a6d', 'value': ''}], 'code': [{'id': 'oehhc-35s-3yd', 'value': 'b'}], 'location': [{'id': 'oehvk-35s-27t1', 'value': ''}], 'childNum': [{'id': 'oeio0-35s-29dx', 'value': '1'}]}, 'id': 4096, 'type': 'vertex', 'label': 'vertex'}
        {'properties': {'_key': [{'id': 'oejuo-6bk-sl', 'value': '15'}], 'functionId': [{'id': 'oelfk-6bk-28lh', 'value': '3'}], 'type': [{'id': 'oek8w-6bk-1l1', 'value': 'AdditiveExpression'}], 'isCFGNode': [{'id': 'oem80-6bk-2a6d', 'value': ''}], 'code': [{'id': 'oekn4-6bk-3yd', 'value': 'a + b'}], 'location': [{'id': 'oel1c-6bk-27t1', 'value': ''}], 'childNum': [{'id': 'oelts-6bk-29dx', 'value': '1'}]}, 'id': 8192, 'type': 'vertex', 'label': 'vertex'}
        {'properties': {'_key': [{'id': 'oen0g-9hc-sl', 'value': '17'}], 'functionId': [{'id': 'oeolc-9hc-28lh', 'value': '3'}], 'type': [{'id': 'oeneo-9hc-1l1', 'value': 'Identifier'}], 'isCFGNode': [{'id': 'oepds-9hc-2a6d', 'value': ''}], 'code': [{'id': 'oensw-9hc-3yd', 'value': 'b'}], 'location': [{'id': 'oeo74-9hc-27t1', 'value': ''}], 'childNum': [{'id': 'oeozk-9hc-29dx', 'value': '1'}]}, 'id': 12288, 'type': 'vertex', 'label': 'vertex'}

Compare this with the output from octopus-shell, where a string representation of the result is returned. What this all means is explained in the section XXX.



