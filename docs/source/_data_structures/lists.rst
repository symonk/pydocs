Data Structures: Lists
======================

Lists: Introduction
--------------------

Python lists are mutable ``sequences`` that store ordered heterogenerous data.  There no concept of duplicates
in a list and both ``hashable`` and ``non hashable`` elements can be stored.

Lists: Instantiation
---------------------

There are three main ways to create a list in python:

    * the `list()` built in.
    * the ``[..., ...]`` square bracket syntax.
    * list comprehensions, ``[char.upper() for char in "foobar"]`` -> `['F', 'O', 'O', 'B', 'A', 'R']`

When creating a simple list, the fastest way to build one is using the `[]` syntax over the `list()` builtin.
This is because there is one less function lookup/call as demonstrated by the following bytecode, a brief benchmark
of the `list()` vs `[]` is outlined below:

    .. code-block:: python

        import dis

        dis.dis("[]")
        """
        1   0 LOAD_NAME                0 (list)
            2 LOAD_NAME                1 (items)
            4 CALL_FUNCTION            1
            6 RETURN_VALUE
        """

        dis.dis("list()")
        """
        1     0 LOAD_NAME                0 (list)
              2 LOAD_NAME                1 (items)
              4 CALL_FUNCTION            1
              6 RETURN_VALUE
        """

        import timeit
        timeit.timeit("[]", number=10_000_000)
        # 0.15199436400143895
        timeit.timeit("list()", number=10_000_000)
        # 0.549999200997263

The reality / performance is often a non factor, but to be as python as possible,
avoid using `list()` when `[]` is an option.  List comprehensions are another beast
altogether which we will discuss later, but for now her is a simple example:

    .. code-block:: python

        x = range(1, 11)
        odd_nums = [n for n in x if n % 2 == 0]
        print(odd_nums)
        # [2,4,6,8,10]

Lists: MRO & Collections
-------------------------

As we previously touched on, `list` types in python are `MutableSequences`, what does this actually mean?  Firstly
let's derive all the subclassing (and virtual subclassing that) is occurring for the python list type.  In order
to achieve that, we can use this handy function:

    .. code-block:: python

        import collections.abc
        import inspect

        def build_hierarchy(col):
            abcapi = vars(collections.abc).items()
            return [v for k,v in abcapi if inspect.isclass(v) and issubclass(col, v)]

        build_hierarchy(list)
        """
        [collections.abc.Iterable,
         collections.abc.Reversible,
         collections.abc.Sized,
         collections.abc.Container,
         collections.abc.Collection,
         collections.abc.Sequence,
         collections.abc.MutableSequence]
        """

We can break down the list inheritance hierarchy (explicit and virtual subclasses) outlined above. As we mentioned,
python lists are `Mutable` and `Sequences`, let's understand what each of these classes offer the list class:

    * ``MutableSequence``: Read-write capabilities on top of ``Sequence``.
        * __setitem__
        * __delitem__
        * insert()
        * append()
        * clear()
        * reverse()
        * extend()
        * pop()
        * remove()
        * Augmented add += via ``__iadd__``
    * ``Sequence``: Operations for read only sequences.  ``class Sequence(Reversible, Collection)``
        * index()
        * count()
        * __getitem__
        * ``__iter__`` required by ``Iterable`` since ``Reversible`` has no implementation of it.
        * __reversed__ as required by its ``Reversible`` parent class
        * __contains__ as required by its ``Collection`` parent class
    * ``Collection`` which itself extends ``Sized, Iterable, Container``.
        *