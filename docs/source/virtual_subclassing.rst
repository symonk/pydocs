Virtual Subclassing
====================

Python Virtual subclassing
---------------------------

Unlike some other languages, python supports the concept of ``virtual subclassing``.  What exactly
is virtual subclassing?  To understand the concept let's go back to pythons roots.  I am sure
you are familiar with the concept of `duck typing` and perhaps if you have came from another
language such as ``java``, you will be familiar with the concept of an ``interface``.  In python
we are not overly caught up on the `type` of `X`, but moreso how `X` behaves, in the simplest
form if it walks like a duck and quacks like a duck, chances are its a good enough duck!

This is best understood by way of example, often times certain objects may share a similar
interface, but do not share a ``is-a`` relationship, let's take the very common idea of
`flying`.  Below we define two completely different concepts that can `fly()`.

    .. code-block:: python

        class Aeroplane:
            def fly():
                print("Boeing in flight...")

        class Bird:
            def fly():
                print("Bird in flight...")


Well, all of these instances can _fly_, however they have no actual relationship.  This is where `collections.abc`
and virtual subclassing can go a long way to helping, virtually registering subclasses allows `subclass` and `instance`
checks treat objects as other subclasses/instances of a type, even if they do not directly subclass that type.  Let's
expand on our earlier example by introducing a