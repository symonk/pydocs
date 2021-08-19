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

