Context Managers
========================

Context Managers: Introduction
-------------------------------

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
    * ``__enter__`` and ``__exit__`` dunder implementations in our own user defined classes.#


Context Managers: User Defined Classes
---------------------------------------

We touched briefly on the dunder ``__enter__`` and ``__exit__`` methods that can be used to turn a
class into a context manager.

 * ``__enter__(self)``

Enters the ``runtime context`` and returns either ``self`` or another object _related_ to the ``runtime
context``.  The ``with`` statement will bind this return value to the target specified in the ``as``
statement, a simple example:

    .. code-block:: python

        from __future__ import annotations
        from typing import Tuple


        class Klazz:
            def __enter__(self) -> Klazz:
                print("Entering the runtime context...")
                return self

        class Klazz2:
            def __enter__(self) -> Tuple[int, int, int]:
                print("Entering the runtime context & returning something else")
                return 100, 200, 300


        with Klazz() as k:
            ...

        with Klazz2() as a, b, c:
            ...

    * ``__exit__(self, exc_type, exc_value, traceback)``


