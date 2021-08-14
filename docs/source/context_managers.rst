Python Context Managers
========================

Context Managers: Introduction
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Python context managers are what underpin the ``with`` statement in python and they are used
for managing resources which need to be closed and or cleaned up.  The typical example with
any introduction to context manager in python is the open statement, to avoid leaving files
open unnecessarily, python builtins support the following:

    .. code-block:: python

        with open("myfile.txt", mode="w+") as file:
            file.write("Hi there\n")

In order to create our own user defined context managers, there are typically two simple approaches, we will
discuss both options in depth throughout this post:

    * ``contextlib.contextmanager`` (as a decorator around a generator function (yield).
    * ``__enter__`` and ``__exit__`` dunder implementations in our own user defined classes.