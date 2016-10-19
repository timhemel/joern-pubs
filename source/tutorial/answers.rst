Answers to exercises
====================


:doc:`nodes_properties_and_traversals`

1.

        .. code-block:: groovy

                g.V()
                .has('type','IfStatement')

2.

        .. code-block:: groovy

                g.V()
                .has('type','IfStatement')
                .values('code')


3. If we evaluate the traversal, we see multiple (two) outputs for a single input, which makes this a ``flatMap`` step.

:doc:`vertex_steps_and_edges`

1. 
        First, we get the file node:

        .. code-block:: groovy

                g.V().has('type','File').has('code',textRegex('.*vertex_steps_and_edges.c'))

        Then, we can ask about the edges, both incoming and outgoing:

        .. code-block:: groovy

                .bothE()

        This gives the following result:

        .. code-block:: json

                [
                {"id": "ppa-6i8-2i39-9k8", "inV": 12392, "inVLabel": "vertex", "label": "IS_FILE_OF", "outV": 8432, "outVLabel": "vertex", "properties": {"var": ""}, "type": "edge"},
                {"id": "ra6-6i8-2i39-cqo", "inV": 16512, "inVLabel": "vertex", "label": "IS_FILE_OF", "outV": 8432, "outVLabel": "vertex", "properties": {"var": ""}, "type": "edge"},
                {"id": "sgu-6i8-2i39-fu0", "inV": 20520, "inVLabel": "vertex", "label": "IS_FILE_OF", "outV": 8432, "outVLabel": "vertex", "properties": {"var": ""}, "type": "edge"}
                ]
                
        We only see outgoing edges with label ``IS_FILE_OF``, so let's follow those, by replacing ``bothE()`` with:

        .. code-block:: groovy

                .out('IS_FILE_OF')

        This gives us a list of nodes. Let's look at one to see its properties:

        .. code-block:: json

                {
                    "id": 12392,
                    "label": "vertex",
                    "properties": {
                        "_key": [
                            {
                                "id": "74d-9k8-sl",
                                "value": "5"
                            }
                        ],
                        "childNum": [
                            {
                                "id": "93h-9k8-29dx",
                                "value": ""
                            }
                        ],
                        "code": [
                            {
                                "id": "7wt-9k8-3yd",
                                "value": "function1"
                            }
                        ],
                        "functionId": [
                            {
                                "id": "8p9-9k8-28lh",
                                "value": ""
                            }
                        ],
                        "isCFGNode": [
                            {
                                "id": "9hp-9k8-2a6d",
                                "value": ""
                            }
                        ],
                        "location": [
                            {
                                "id": "8b1-9k8-27t1",
                                "value": "3:0:8:38"
                            }
                        ],
                        "type": [
                            {
                                "id": "7il-9k8-1l1",
                                "value": "Function"
                            }
                        ]
                    },
                    "type": "vertex"
                }

        Now we can see the name of the function in the ``code`` property. The
        final query now becomes:

        .. code-block:: groovy

                g.V()
                .has('type','File')
                .has('code',textRegex('.*vertex_steps_and_edges.c'))
                .out('IS_FILE_OF')
                .values('code')


:doc:`files_and_functions`

1.

        We can label intermediate results of a traversal for later use. This is done
        with the ``as()`` step. The intermediate results can later be used in the
        traversal through the ``select()`` step:

        .. code-block:: groovy

                traversal1().as('trav1')
                .traversal2().as('trav2')
                ...
                .select('trav2','trav1')

        The traversal in the exercise finds functions whose name ends in ``sample``,
        labels it as ``func``, then finds the file in which it is defined, labels that
        as ``file``, and then combines both ``func`` and ``file`` in the result.


:doc:`ast_traversals`

1.
        Function calls have a node type of ``CallExpression``. We can use
        a simple filter traversal as an argument to ``emit``:

        .. code-block:: groovy

                .emit( has('type','CallExpression') )

2.
        If I run the traversal, I see a breadth first traversal order:

        .. code-block:: none

                [274560, FunctionDef, tut1 ()]
                [352360, Identifier, tut1]
                [393264, ReturnType, void]
                [675928, CompoundStatement, ]
                [708696, ParameterList, ]
                [315488, ExpressionStatement, print ( x )]
                [385072, IfStatement, if ( z > 0 )]
                [680024, IdentifierDeclStatement, int x = 1 ;]
                [688216, ExpressionStatement, print ( x )]
                [348408, CallExpression, print ( x )]
                [339968, CompoundStatement, ]
                [692312, Condition, z > 0]
                [336120, IdentifierDecl, x = 1]
                        :              :

        However, the order cannot be guaranteed, according to the `Tinkerpop3 manual <http://tinkerpop.apache.org/docs/3.0.1-SNAPSHOT/#_the_traverser>`_:

                A Traversal’s result are never ordered unless explicitly by means of order()-step. Thus, never rely on the iteration order between TinkerPop3 releases and even within a release (as traversal optimizations may alter the flow).


3a.
        We can start with the simple repetition that gives us all ``Identifier`` nodes:

        .. code-block:: groovy

                .emit(has('type','Identifier'))
                .repeat(
                        out('IS_AST_PARENT')
                )
                .values('code')
                        
        If we run this on the ``FunctionDef`` node however, this will give us
        function names and parameter names as well. These are not variables.

        Assuming that we will not store functions in variables and call them, we can eliminate ``Identifier`` nodes that descend from ``Callee`` nodes, by adding a filter step inside the repetition:

        .. code-block:: groovy

                .repeat(
                        out('IS_AST_PARENT')
                        .has('type',P.without('Callee'))
                )

        The Identifiers that are used directly in the ``FunctionDef`` node
        should also be eliminated. However if we filter out ``FunctionDef``
        nodes, we would not get any real variables either!

        We could add a traversal to the ``emit`` step.

        .. code-block:: groovy

                .emit(has('type','Identifier').in('IS_AST_PARENT').has('type',P.without('FunctionDef')))
                .repeat(
                        out('IS_AST_PARENT')
                        .has('type',P.without('Callee'))
                )
                .values('code')

        This would get the job done, more or less, but it feels like we are
        doing too much work.

        Another method is to put the condition inside the repetition, using the ``coalesce`` step:

        .. code-block:: groovy

                .emit(has('type','Identifier'))
                .repeat(
                        coalesce(
                                has('type',P.within('Callee','FunctionDef'))
                                .out('IS_AST_PARENT')
                                .has('type',P.neq('Identifier')),
                                has('type',P.without('Callee','FunctionDef'))
                                .out('IS_AST_PARENT')
                        )
                )

        It looks like there is no easy way to escape the extra work! The real
        cause of the problem here is of course that the parser that generated
        the AST does not distinguish between identifiers that refer to a
        variable or identifiers that refer to a function name.

        There is one last step, and that is to eliminate duplicate results.
        For this, we can add the ``dedup`` step at the end:

        .. code-block:: groovy

                .emit(has('type','Identifier').in('IS_AST_PARENT').has('type',P.without('FunctionDef')))
                .repeat(
                        out('IS_AST_PARENT')
                        .has('type',P.without('Callee','FunctionDef'))
                )
                .values('code')
                .dedup()

3b.
        In this case, a variable is changed if it is on the left-hand side
        of an ``AssignmentExpression``. We further assume that this left-hand
        side contains a variable without any further ado, i.e. no array or
        pointer dereferences.

        We start by finding the ``AssignmentExpressions``:

        .. code-block:: groovy

                .emit(has('type','AssignmentExpression'))
                .repeat(
                        out('IS_AST_PARENT')
                )

        The left-hand side of an assignment expression is the AST child that
        has a ``childNum`` property equal to ``0``. Note that the childNum
        attribute has a string and not a number type. We traverse to the AST
        children and select the node with ``childNum`` equal to ``'0'``. At
        the end, we ``dedup`` the results:

        .. code-block:: groovy

                .out('IS_AST_PARENT')
                .has('childNum','0')
                .values('code')
                .dedup()

:doc:`defining_custom_gremlin_traversals`

1.

        a.

                If we add the following print statements:

                .. code-block:: groovy

                        addStep("getarginfo", { a ->
                                println a
                                println a.getClass()
                                println a.dump()
                                delegate
                        })

                we see the following in the console:

                .. code-block:: none

                        [2]
                        class [Ljava.lang.Object;
                        <[Ljava.lang.Object;@4ecb9891>

                This means that the argument ``a`` is a list of
                ``java.lang.Object``.

        b.

                This is a possible solution:

                .. code-block:: groovy
        
                        addStep("nthChildren", { args ->
	                        delegate.out('IS_AST_PARENT').has('childNum',args[0])
                        })


        c.
                The code can be found in the file ``ast.groovy``.

