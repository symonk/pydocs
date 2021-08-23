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