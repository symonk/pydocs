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

Sets: Operations & Set Theory
------------------------------

...

Sets: Summary (tldr)
---------------------

    * :class:`frozenset` are immutable, :class:`set` are mutable.
    * :class:`set` contain unordered, non indexable distinct hashable immutable elements.
    * Using empty `set comprehension` syntax will actually generate a `dictionary`.
    * create :class:`set` using `set()`, `{1,2,3}` or `{n for n in range(10) if n % 2 == 0}`.
    * create :class:`frozenset` using the `frozenset()` callable.
    * user defined objects can be stored in sets by default, but are never considered equal.
    * to add user defined objects to sets, implement `__hash__` and `__eq__`.