More Side-effects: Using Sacks
==============================

Sometimes we need to keep information around during a traversal, for example
to compute something or to do some bookkeeping. In Gremlin, you would typically
use side-effect steps for this. We talked about collecting objects using ``aggregate`` or ``store``, and to use ``sideEffect`` for print debugging.

A more generic method is the use of *sacks*. A sack is a data structure that is attached to a traverser. Using the ``sack`` step, you can read and write to this data structure. The `Gremlin documentation <http://tinkerpop.apache.org/docs/3.0.1-SNAPSHOT/#sack-step>`_ explains this well.

Here we present two extra examples, that operate on the CFG.

Which nodes define a variable
-----------------------------

In the section on CFG traversals, we have looked at how to find the variables that are defined in each node, but now we would like to know in which nodes a certain variable is defined. We can solve this with a sack.

We want to compute a map that tells us which nodes assign to a certain variable, in other words, a map that maps a variable name into a list of nodes.
In order to use sacks, we need to tell that we will use a sack at the beginning
of the traversal:

.. code-block:: groovy

        g.withSack([:])
        .V()
        .has('type','Function').has('code','tut5')

The argument of ``withSack`` is stored in the traverser. If it is a simple value, it is stored as a value, but if it is complex object, such as the ``Map`` object in this case, a reference is used. This means that you could run into aliasing problems, or even worse, race conditions if the implementation parallellizes traversals. One solution is to ``clone()`` the object, which will store a copy of the sack value in each traverser. In this example we have not done that.

To modify the sack in a traversal, we use the ``sack`` step. Without any arguments, it is a map step that will simply map the current object in the traverser to the value of the sack. Given a function (or closure) however, it is a side-effect step that will update the sack value. The function or closure needs to have a specific signature, following a monad pattern. It takes both the sack value and the current object as arguments and returns the (updated) sack value. If we fit this in the CFG traversal, we get:

.. code-block:: groovy

        .functionToStatements()
        .as('cfgnode')
        .definedVariables()
        .as('var')
        .select('cfgnode','var')
        .sack{ m,obj ->
                var = obj['var'] ;
                node = obj['cfgnode'] ;
                m[var] = (m.get(var,[]) + [node] );
                m
        }

The nodes and variables that are defined in the CFG nodes are first collected into a mapping. Then the sack is updated by updating the map. This is Groovy code that adds a node to the list of nodes of a certain variable, or to the empty list if that variable does not have a node list yet. And we should not forget to return the updated sack value (note that an explicit ``return`` statement is not needed in Groovy).

We could now use ``sack`` without arguments to get the sack, but then we would get multiple values of the sack, some of which may contain intermediate results. To solve both problems, we add a ``fold`` step to collect all traversal results into one traversal result, and then use the ``sack`` step.

.. code-block:: groovy

        .fold()
        .sack()


Counting the number of visits
-----------------------------


In data flow analysis we often need to iteratively compute properties of CFG nodes. If this is done within a CFG traversal, it means that we need to visit a CFG node multiple times. Sometimes a bound is given on the number of times that a CG node can be visited.

In Gremlin, the traverser has a ``loops`` method, which will return how many loops the traverser has encountered in the entire traversal so far. This is not exactly what we want.

We could implement this by keeping track of the number of visits in the node itself, but this would cause problems if multiple traversals would happen in parallel on the same nodes. Furthermore, storing data in vertex properties is relatively expensive and not intended for volatile information such as counters.

The sack does not suffer from such problems: the sack is local to the traverser and is intended especially for volatile information. To keep the count per node, we will need a ``Map``, just like in the previous example. This time however, the count must be kept *per traversal*, which means that we must clone the sack.
This is done at the start:

.. code-block:: groovy

        g.withSack([:]){ it.clone() }
        .V()
        .has(NODE_TYPE,TYPE_FUNCTION)
        .has(NODE_CODE,'tut5')

The second argument to ``withSack`` is the so-called 'split operator' and is executed when the traversal is split into more traversals. This ensures that we are always working on a unique copy of the sack in each traversal. Next, we traverse to the CFG entry node so that we can start our loop.

.. code-block:: groovy

        .functionToCFG()             // gives the CFG entry node

The loop needs to terminate the traversal if the number of visits to the current node is more than say, three. This termination can be ensured by adding an ``until`` step. In the repetition, we update the step in a similar way as in the earlier example:

.. code-block:: groovy

        .until{ it.sack.get(it.get().id(),0) >= 3 }
        .repeat(
                sack{ m,v -> m[v.id()] = (m.get(v.id(),0)+1); m }
                .out(CFG_EDGE)
        )
 
A disadvantage of the ``until`` step is that if the condition holds, the traversal is seen as succesful and the traversal will continue with the current object (in other words, it will be 'emitted'.) This is not always what you want. Another way to stop traversing if a CFG node has been visited three times or more, is to filter out those traversals using a ``filter`` step:

.. code-block:: groovy

        // no until() step
        .repeat(
            filter{ it.sack.get(it.get().id(),0) < 3 }
            .sack{ m,v -> m[v.id()] = (m.get(v.id(),0)+1); m }
            .out(CFG_EDGE)
        )

This will not emit any nodes and you can insert whatever computation that you like. 


# POPA 2.1.2

Exercises
-----------

1.

        Write a traversal that outputs all code paths in which a node is
        visited at most three times. Use the ``path`` step.


