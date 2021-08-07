Iterator Protocol
=========================

Python Iterator Protocol
-------------------------

Here we out cover the iterator protocol in depth, both the newer version via
``__iter__`` and ``__next__`` as well as the older protocol piggy backing
off ``__getitem__` for sequence types.

In python if you have found yourself wondering, how does the `for each` loop
work?  This is where the iterator protocol comes into play and making your
own iterable user defined types is surprisingly straight forward.  Personally
the terminology is more complex than the actual logic involved.  Let's
try and break it down:

    * ``collections.abc.Iterator`` extends ``collections.abc.Iterable``
    * All ``iterators`` are ``iterable``
    * Not all ``iterables`` are ``iterators``

Iterator Protocol: Iterable ABC
--------------------------------

``collections.abc.Iterable`` is an abstract base class built into python
that offers up the abstract ``__iter__`` method that should be implemented.
Rule of thumb is that anything that is ``iterable`` when asked for an
``iterator`` will return one.

    .. code-block:: python

        @abstractmethod
        def __iter__():
            while False:
                yield None

That's it, iterables are really that simple, if something is iterable, it can
return an ``iterator``.  It is an ``iterator` that python actually uses to
perform iteration.  The built in ``iter()`` function calls an objects
dunder ``__iter__``.

Iterator Protocol: Iterator ABC
--------------------------------


