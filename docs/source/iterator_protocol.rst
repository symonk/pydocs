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

Iterator Protocol: Iterables
-----------------------------

