Virtual Subclassing
====================

Python Virtual subclassing
---------------------------

Unlike some other languages, python supports the concept of ``virtual subclassing``.  What exactly
is virtual subclassing?  To understand the concept let's go back to pythons roots.  I am sure
you are familiar with the concept of `duck typing` and perhaps if you have came from another
language such as ``java``, you will be familiar with the concept of an ``interface``.  In python
we are not overly caught up on the `type` of `X`, but moreso how `X` behaves, in the simplest
form if it walks like a duck and quacks like a duck, chances are its a duck.  Python does not support
an official ``interface`` argument, however more recently ``Protocols`` can be ``@runtime_checkable``
to aid with catching issues early.

To understand why we may need virtual subclassing, let's take a simple example.  Firstly we are tasked
with developing an airplane tycoon game.  We develop an abstract base class which implements a `fly()`
method for our planes to subclass and implement.

    .. code-block:: python

        from abc import ABC
        from abc import abstractmethod

