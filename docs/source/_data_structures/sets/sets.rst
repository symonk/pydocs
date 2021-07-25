Python Data Structures: Sets
============================

Python sets are:

    - Unordered, non indexable, distinct immutable (hashable) elements.
    - Come in two flavours :class:`set` (mutable) & :class:`frozenset` (immutable).
    - Sets cannot contain other sets as they are not hashable, they can contain :class:`frozenset` instances.
    - Sets offer quick membership testing `in` and removing duplicates from other collections.
    - Sets support a whole host of mathematical operations (set theory) such as `union` & `intersection` etc.

-----

Sets: Instantiation
--------------------

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
        # TypeError: un-hashable type: `set`
        frozen = frozenset({1,2})
        >>> frozenset({1,2})

More can be found about frozenset later in the documentation.

We touched briefly on sets being unable to add non hashable elements, in python both
:class:`list` and :class:`dict` are also `mutable` and thus, neither can be added
to a normal :class:`set`:

    .. code-block:: python

        s = {[1,2,3]}
        # TypeError: un-hashable type: `list`
        s = {dict(a=1)}
        # TypeError: un-hashable type: `dict`

However, because the `set()` class permits building a set from an iterable and both
list and dictionary are iterable (dict over keys by default), then populating a set
from both of the collections is possible:

    .. code-block:: python

        s = set([1,2,3,4,5])
        # {1, 2, 3, 4, 5}
        s = set(dict(a=1, b=2, c=3))
        # {'a', 'b', 'c'}

Sets: Distinction
------------------

We mentioned previously that sets must contain hashable elements only, this is because similarly to
dictionary keys, sets use the hash value of the object it is attempting to store internally.  This
is why `in` checks are extremely fast in sets, they are backed by a hash table. In order to be able
to store your custom objects in a `set` (or alternatively use them for dictionary `keys`) you can
implement two magic methods, `__hash__` and `__eq__` respectively.

By default, user defined objects have the following in python:

    * an implementation of `__hash__`.
    * an implementation of `__eq__` which results in no two instances being equal.

    .. code-block:: python

        class Example:
            def __init__(self, x: int) -> None:
                self.x = x

        e = Example(100)
        e2 = Example(100)
        hash(e)  # 108032011057
        hash(e2)  # 108032014237 (different)
        e == e2  # False
        {e, e2}  # {<__main__.Example at 0x192735ab310>, <__main__.Example at 0x192735b79d0>}

By default this permits us to store instances of `Example` in a set by default as highlighted above.
In order to use our own user defined objects in sets effectively, we should implement both the
dunder `__hash__` and `__eq__` methods to consider two instances of :class:`Example` equal.

    .. code-block:: python

        from __future__ import annotations # __eq__ `other` type hint of the class itself

        class ImprovedExample:
            def __init__(self, x: int) -> None:
                self.x = x

            def __hash__(self) -> int:
                return hash(self.x)

            def __eq__(self, other: ImprovedExample) -> bool:
                # note: returning `NotImplemented` here tells python to try the reflected operation on `other`.
                if not isinstance(other, type(self)): return NotImplemented
                return self.x == other.x

Now we are able to store instances of `ImprovedExample` in both sets and in dictionaries as keys:

    .. code-block:: python

        one, two, three = ImprovedExample(100), ImprovedExample(200), ImprovedExample(100)
        {one, two, three}  # one == three & hash(one) == hash(three) thus only 2 are stored (distinct)
        """
        {<__main__.ImprovedExample at 0x1927465c490>,
        <__main__.ImprovedExample at 0x1927465c880>}
        """

** If a class does not implement dunder __eq__, it should never implement dunder __hash__. **

Sets: Method resolution order
------------------------------
Pythons :class:`collections.abc.Set` MRO is described below:

    .. code-block:: python

        from collections.abc import Set

        Set.mro()
        """
        (collections.abc.Set,
         collections.abc.Collection,
         collections.abc.Sized,
         collections.abc.Iterable,
         collections.abc.Container,
         object)

        Set inherits from `Collection`
        `Collection` inherits from `Sized` which provides len(set).
        `Collection` inherits from `Iterable` which allows sets to be iterated over.
        `Collection inherits from `Container` which allows sets to perform `in` checks via `__contains__`.
        and lastly, everything inherits from `object`.

        `Set` inherits a lot of additional capabilities through its mixin methods:
            * __le__
            * __lt__
            * __eq__
            * __new__
            * __gt__
            * __ge__
            * __and__
            * __or__
            * __sub__
            * __xor__
            * isdisjoint()

        A lot of these mixin methods will be discussed later in depth and how objects
        can slot right into pythons data model and be considered pythonic.
        """

Sets: Operations (Basics)
--------------------------

Many operations supported on other data structures do not make logical sense for sets,
however sets themselves offer a very robust set of operations to align them nicely
with sets in mathematics.  Some functionality not supported by sets are (that of sequences)
like slicing a set, or finding the `index` of a given `element` within the set.

    .. code-block:: python

        s = {1,2,3,4,5,6}
        s[1:3]
        # TypeError: set object is not subscriptable

        s = {5,4,3,2,1}
        s.index(4)
        # AttributeError: set object has no attribute: index

In order to fully understand the power of sets, we need to understand the distinct
differences between three things:

    * object methods
    * object operations
    * augmented operations  (we will touch on this later on).

Almost all the functionality of python sets can be performed in two main ways.  Via
set instance methods, for example:

    .. code-block:: python

        s = {1,2,3}
        s.union({3,4,5})  # Method invocation -> {1,2,3,4,5}


Alternatively, as we touched on earlier, through various mixin methods implemented on
:class:`Set`, the following is also supported:

    .. code-block:: python

        one = {1,2,3}
        two = {3,4,5}
        one | two  # Operation invocation -> {1,2,3,4,5}

Notice how the duplicate `3` entry in both cases is deduped, a simple trait of sets (to remove
duplicates).  Both examples above result in (almost) the same thing happening, functionally it
is the same, however operations tend to be slightly faster, this is outlined below:

    .. code-block:: python

        import dis
        one = {1,2,3}
        two = {3,4,5}
        dis.dis("one.update(two)")
        """
        1     0 LOAD_NAME                0 (one)
              2 LOAD_METHOD              1 (update)
              4 LOAD_NAME                2 (two)
              6 CALL_METHOD              1
              8 RETURN_VALUE
        """

        dis.dis("one | two")
        """
          1   0 LOAD_NAME                0 (one)
              2 LOAD_NAME                1 (two)
              4 BINARY_OR
              6 RETURN_VALUE
        """

For a real world bench mark, lets perform the same task (getting the union of the above two sets) to
see the difference (20 million times).

    .. code-block:: python

        import timeit
        timeit.timeit("one.union(two)", setup="one={1,2,3}; two={3,4,5}", number=20_000_000)
        # 4.246593700000005 (4.2 seconds)
        timeit.timeit("one | two", setup="one={1,2,3}; two={3,4,5}", number=20_000_000)
        # 3.168324699999971 (3.1 seconds)

While neglible it is important to understand that operator approaches are often faster.  There are however
a few subtle differences / caveats to be aware of.

    * when using the method based approach, e.g `union()` any `iterable` can be provided and python will handle it
    * when using the operator based approach, e.g `|` all objects must be of type: `set`.

    .. code-block:: python

        s = {1,2,3}
        s.union([2,4,6,8])
        # {1, 2, 3, 4, 6, 8}
        s | [2,4,6,8]
        # unsupported operand type(s) for |: `set` and `list`.

By default, both the methods and basic operators return a new `set` instance.  We briefly spoke about
`augmented operators`, these can be used to modify set `s` in-place, more on that later.


Sets: Operations (Advanced)
----------------------------


Sets: Summary
--------------

    * :class:`frozenset` are immutable, :class:`set` are mutable.
    * :class:`set` contain unordered, non indexable distinct hashable immutable elements.
    * Using empty `set comprehension` syntax will actually generate a `dictionary`.
    * create :class:`set` using `set()`, `{1,2,3}` or `{n for n in range(10) if n % 2 == 0}`.
    * create :class:`frozenset` using the `frozenset()` callable.
    * user defined objects can be stored in sets by default, but are never considered equal.
    * to add user defined objects to sets, implement `__hash__` and `__eq__`.
    * :class:`Set` inherits from :class:`collections.abc.Collection` which in turns inherits from `Sized`, `Iterable`, `Container`.
    * :class:`Set` permits many of its functionality through both method calls and operators.
    * :class:`Set` operator usage tends to be slightly faster due to not having to load & call a method.