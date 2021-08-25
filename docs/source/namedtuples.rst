Collections: namedtuple
========================

Namedtuple: Introduction
-------------------------

``collections.namedtuple`` is a simple factory function for building more advanced tuples.  Just
like tuples they permit indexing, are iterable and the main benefit they offer over a standard
tuple is that attributes can be accessed by name.  Here is a basic introduction to named tuples:

    .. code-block:: python

        from collections import namedtuple

        Foo = namedtuple("Foo", "bar, baz", defaults=(100, 200))
        print(Foo())
        #  Foo(bar=100, baz=200)

As you can see from this small snippet, namedtuple also offer a nice dunder `__repr__` implementation
right off the bat.  We will discuss more throughout this article, especially the factory function arguments
in great detail, but for now just know that namedtuples create tuple subclasses with attribute access.

Namedtuple: Factory Function
-----------------------------

The ``namedtuple(...)`` factory function offers a lot of additional arguments, often overlooked and
to be honest, rarely used, however for a full overview, we will discuss each argument and what it does
with an example.

    .. code-block:: python

        collections.namedtuple(
            typename: str,
            field_names: Iterable[str],
            *,
            rename: bool = False,
            defaults: Optional[Any] = None,
            module: Optional[Any] = None
        )

Namedtuple: typename
---------------------
typename is the new tuple subclass name.  In order for pickling to be natively supported
the typename should match the name of the variable assigned to the tuple subclass. This
is briefly shown below:

    .. code-block:: python

        import pickle
        from collections import namedtuple

        Foo = namedtuple("Bar", "a,b,c", defaults=(200,300))
        f = Foo(a=25)
        print(f)
        #  Bar(a=25, b=200, c=300)
        pickle.dumps(f)  #  PicklingError: Can't pickle <class '__main__.Bar'>
        Foo2 = namedtuple("Foo2", "a,b,c", defaults=(200, 300))
        f = Foo2(25)
        fbytes = pickle.dumps(f)
        # b'\x80\x04\x95 \x00\x00\x00\x00\x00\x00\x00\x8c\x08__main__\x94\x8c\x04Foo2\x94\x93\x94K\x19K\xc8M,\x01\x87\x94\x81\x94...."

Namedtuple: field_names
------------------------
``field_names`` is a sequence of strings or an individual string of the attribute names to be
assigned to the underlying tuple subclass.  In the latter, attribute names are automatically
resolved by splitting on either a comma, or whitespace,  named tuples do not have an
underlying ``__dict__`` instance (think ``__slots__``) which is what allows them to compete
with standard tuples on memory.

    .. code-block:: python

        from collections import namedtuple

        f = namedtuple("f", ["one", "two", "Three"])
        f2 = namedtuple("f2", "one two three")
        f3 = namedtuple("f3", "one, two, three")

field names can be any valid python identifier except for anything starting with an underscore.
named tuple has an extra param we will discuss later `rename=` which is used for rewriting
illegal field names automatically with a prefixed underscore.

Namedtuple: rename
-------------------
As previously outlined, `rename` works in tandem with the `field_names` argument in order to
automatically rewrite name violations with a prefixed `_` positional names, where each
violation is incremented += 1.  For example:

    .. code-block:: python

        from collections import namedtuple

        One = namedtuple("One", "one, def, two, class, three, return", rename=True)
        one = One(10, 20, 30, 40, 50, 60)
        #  One(one=10, _1=20, two=30, _3=40, three=50, _5=60)

As you can see in the example, field names `def`, `class` and `return` are also python
core builtin reserved keywords, these have automatically been rewritten with `_<n>` for
each violation in the sequence passed to `field_names`.

Namedtuple: defaults
---------------------
namedtuple defaults is an ``iterable`` of names to unpack into the fields when a value is omitted.
By default, the values are unpacked from `<-` right to left, so if there are three field names
defined `a,b,c` and two defaults `defaults=(100, 200)` then `b` == `100` and `c` == `200`, `a`
is a required field in this instance.  ``defaults=`` can also be ``None`` in which case, all
field_name attributes are `required`.


    .. code-block:: python

        from collections import namedtuple

        Foo = namedtuple("Foo", "a,b,c", defaults=(10, 20))
        f = Foo()
        # __new__() missing 1 required positional argument: 'a'
        f = Foo(2000)
        print(f) #  Foo(a=2000, b=10, c=20)

namedtuple: module
-------------------
namedtuple allows you to customise the `module` of the tuple subclass, if `module=` is assigned
then the dunder ``__module__`` of the namedtuple will be set to that.  ``__module__`` is a writable
field defining the name of the module the function was defined in.  This is shown below (using an
interactive `ipython` shell where by default the module would be ``__main__``.

    .. code-block:: python

        from collections import namedtuple

        T1 = namedtuple("T1", "a,b,c", defaults=(1,2,3), module="foomod")
        T2 = namedtuple("T2", "a,b,c", defaults=(3,2,1))

        # T1 has a custom module name assigned; let's inspect its instances:
        T1().__module__  # foomod
        # T2 omits the module attribute from the sig
        T2().__module__  # __main__


Namedtuple:  misc
------------------
In order for namedtuple instances to be a core part of the python language, they need to retain
some of the benefits of standard tuple types.  Namedtuples do not have a per instance dictionary
(only a class one) this is how they are able to retain the same memory footprint as normal tuples.
They are of course also immutable and in order to support pickling by default, the variable
named assigned to the namedtuple instance should match that of the defined `typename`.  These
are outlined below:

    .. code-block:: python

        # -- Memory Footprint
        from sys import getsizeof
        t = (100, 200, 300)
        nt = namedtuple("Foo", "a b c", defaults=(100,200,300))()  # Create the instance!
        getsizeof(t)  # 64 bytes
        getsizeof(nt)  # 64 bytes

        # -- Immutability
        Immutability = namedtuple("Immutability", "one, two", defaults=(500, 600))
        immutable = Immutability()
        immutable.__dict__  # `Immutability object has no attribute __dict__`
        immutable.one, immutable.two  # (500, 600)
        immutable.one = 2  # AttributeError: Cannot set attribute

        # -- Pickle capabilities
        import pickle
        Works = namedtuple("Works", "a")
        w = Works(10)
        pickle.dumps(w)  # bytes no problem.

        DoesntWork = namedtuple("Different", "a")
        d = DoesntWork(20)
        pickle.dumps(d)  # PicklingError: Can't pickle <class '__main__.Different'>: attribute lookup Different on __main__ failed

Namedtuple: _make
------------------
The first of the three main methods that are bolted onto namedtuple instances.  `_make` is a `@classmethod`. that
uses `tuple.__new__` under the hood to create a new namedtuple instance from an iterable.

    .. code-block:: python

        from collections import namedtuple

        T = namedtuple("T", "a b c", defaults=(100,150, 200))
        t = T()  # T(a=100, b=150, c=200)
        t2 = t._make((5,15,25)) # T(a=5, b=15, c=25)

Namedtuple: _asdict
--------------------
The second of the three main methods that are bolted onto namedtuple instances.  `_asdict` returns a
dictionary of the namedtuple instance attributes and corresponding values.  As of python `3.8` the
`_asdict` function returns a normal dictionary, if you need the benefits of an `OrderedDict` consider
instantiating one directly using this `_asdict` function:

    .. code-block:: python

        from collections import namedtuple
        from collections import OrderedDict

        T = namedtuple("T", "a,b", defaults=(50, 100))
        t1 = T()
        mapping = t1._asdict()
        #  {"a": 50, "b": 100}
        order = OrderDict(t1._asdict())
        #  OrderedDict([('a', 50), ('b', 100)])

Namedtuple: _replace
---------------------
The third of the three main methods bolted onto namedtuple instances is `_replace`.  This allows you
to create a new instance of the tuple subclass, replacing fields of the existing instance with
keys and respective values from the `**kwargs` mapping:

    .. code-block:: python

        from collections import namedtuple
        T = namedtuple("T", "a,b,c", defaults=(12, 24, 36))
        t = T()  # T(a=12, b=24, c=36)
        mapping = {"c": 4000}
        t2 = t._replace(**mapping)
        t2  # T(a=12, b=24, c=4000)

Namedtuple: _fields
--------------------
The `_fields` instance attribute is used for a simple tuple of the namedtuple instance field names.
This is useful for introspection and new namedtuple instances containing a subset of an existing
instances fields, this recipe is outlined below:

    .. code-block:: python

        from collections import namedtuple

        T = namedtuple("T", "a")
        t = T(100)  # T(a=100)
        T2 = namedtuple("T2", t._fields + ("b", "c"), defaults=(50, 60,70))
        t2 = T2()
        t2 # T2(a=50, b=60, c=70)
