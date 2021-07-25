Python Data Structures: Sets
============================

Python sets are:

    - Unordered, non indexable, distinct immutable (hashable) elements.
    - Come in two flavours :class:`set` (mutable) & :class:`frozenset` (immutable).
    - Sets cannot contain other sets as they are not hashable, they can contain :class:`frozenset` instances.
    - Sets offer quick membership testing `in` and removing duplicates from other collections.
    - Sets support a whole host of mathematical operations (set theory) such as `union`, `difference`, `intersection` etc.

-----

Set Instantiation
-----------------

Python sets can be created in a number of different ways:

    .. code-block:: python

        # simple set() constructor:
        empty_set = set()
        # set from any iterable:
        set_from_iter = set(range(1, 10))
        # set using the braces syntax:
        set_braces = {"one", "two", "three"}
        # set using a set comprehension:
        set_comp = {n for n in range(20) if n % 2 == 0}


Care is advised when using the curly braces, often when trying to create an empty set, subtle bugs
can be introduced as python treats `{}` as a :class:`dict`.

    .. code-block:: python

        type({}) # dict

Sets themselves are not immutable and thus, not hashable so this means that sets cannot store sets within
themselves, another build in data structure is the :class:`frozenset` which can be used as elements inside
sets themselves:

    .. code-block:: python

        s = {{1,2}, {3,4}}
        # TypeError: unhashable type: `set`
        frozen = frozenset({1,2})
        >>> frozenset({1,2})

More can be found about frozensets later in the documentation.

