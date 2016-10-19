Defining Custom Gremlin Traversals
===================================

As we are getting more familiar with Gremlin traversals, we may think of
re-using some traversals. In Gremlin, a traversal is nothing more than an
iterator function. Traversals are set up in such a way that they can be
combined, as we have seen in how steps are chained. Fortunately, you do not
need to know the data structure internals to define your own traversals. (If
you are interested however, have fun studying the `Gremlin API
<http://tinkerpop.apache.org/javadocs/3.0.1-SNAPSHOT/full/>`_.

Groovy
------

The Octopus shell uses the `Groovy <http://www.groovy-lang.org>`_ variant of
Gremlin. In other words, a Gremlin script is just a Groovy script, in which
you can use Gremlin objects and functions. For example, you could type
``print "Hello world!\n"`` in the Octopus shell, and the Octopus server would
run that code, printing ``Hello world!`` to the console from which the Octopus
server was started.

Closures
--------

One advantage of Groovy over Java is that we do not have to be very strict with
our types. Groovy will figure out what type is meant, if possible. This is used
heavily in so-called *closures*, which are a bit like anonymous functions.

Many Gremlin steps accept a closure as an argument, such as for example the
``emit`` step. Instead of:

        .. code-block:: groovy

                .emit(has('type','AssignmentExpression'))

we could have written:

        .. code-block:: groovy

                .emit{ it.get().value('type') == 'AssignmentExpression' }

Note the ``{ ... }`` braces instead of the parentheses. Whatever is between
curlies, is interpreted as a list of statements. The value of the last
statement expression will be the return value of the closure.
But if closures are like anonymous functions, then how can we use arguments
that are passed to it? Arguments to a closure can be used as follows:

        .. code-block:: groovy

                { arg1, arg2, ... -> statement1 ; statement2 ; ... }

Closures already have implicit parameters. The ``it`` parameter is always
available. When used in a closure as argument to a Gremlin step, the ``it``
parameter denotes the current ``Traverser``. This is not the object that is
currently processed, but more like a bookkeeper that keeps track of everything
during the traversal. To get to the object that is processed, we use ``it.get()``. More details on closures can be found in the `Groovy language documentation <http://www.groovy-lang.org/closures.html>`_.

Closures can be really useful when debugging a traversal. The ``sideEffect``
step, with a closure as an argument can be used for `print debugging <https://en.wikipedia.org/wiki/Debugging#Techniques>`_. An example:

        .. code-block:: groovy

                .emit()
                .repeat(
                        sideEffect{ l =  [ it.get().id(), it.get().value('type'), it.get().value('code') ]; println l.join(" ") }
                        .out('IS_AST_PARENT')
                )

This will print debug information to the server console and allows us to see
in what order the AST nodes are traversed:

        .. code-block:: none

                        :     :
                290944 Callee print
                700504 ArgumentList x
                348264 ExpressionStatement x = 2
                696408 RelationalExpression z > 0
                        :     :

Defining a Traversal
--------------------

The Octopus shell defines a function ``addStep`` that allows you to define a
traversal. For example, if we want to get the right-hand side of an equality
or an assignment, we can define a traversal as follows:

        .. code-block:: groovy

                addStep("rval", {
                        delegate.out(AST_EDGE).has(NODE_CHILDNUM, '1')
                })

This traversal is defined in the standard library that is loaded in the
Octopus shell. The first argument to ``addStep`` is the name of the traversal,
the second one is a closure. In this case, the closure uses the implicit
variable ``delegate``. If you are interested in the details, read the
`Groovy language documentation <http://www.groovy-lang.org/closures.html#closure-owner>`_, otherwise just remember to use ``delegate`` when you define a
new traversal.

We can then use the traversal as if it were a normal Gremlin step:

        .. code-block:: groovy

                out('IS_AST_PARENT')
                .has('type','AssignmentExpression')
                .rval()

Although at the implementation level, steps and traversals are different, we
will sometimes use 'step' to refer to user-defined traversals, because for our
purposes, they can be used just like a Gremlin step.

The standard library
---------------------

TODO: location, where to add steps, etc.


Exercises
---------

1.

        a.

                We have the following traversal definition that takes an
                argument and does nothing. We have also added code that
                uses it:

                .. code-block:: groovy
        
                        addStep("getarginfo", { a ->
                                delegate
                        })

                        g.V().getarginfo(2)

                Through print debugging, find out the type of ``a``. You may
                be interested in the ``getClass()`` and ``dump()`` methods
                that are available on every Groovy object.


        b.

                Write a traversal that gives AST child number *n* for an AST node.
                That is, if I use

                .. code-block:: groovy

                        nthChildren('2')

                I will get the node for which the ``childNum`` property is ``'2'``.

        c.

                Look for the definition of ``ithChildren`` in the standard
                library and compare your solution to it.


