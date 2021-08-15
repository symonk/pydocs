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

Dunder exit as you could have guessed, is responsible for ``closing`` a resource, clean up after the core
with block has been executed, called implicitly by python.  There are a few things to know about dunder
``__exit__`` and we will discuss those here as well as some of the caveats of incorrectly implementing it.

Dunder ``__exit__`` exits the runtime context and in provides exception information for any exceptions
unhandled during the runtime context, if no exceptions where raised during the runtime context, then
all parameters passed to __exit__ will be ``None``.  This is outlined below:

The parameters passed in to handle the exception information is as follows:

    * ``exc_type`` -> The ``type`` of the exception class raised.
    * ``exc_value`` -> The exception value, e.g -> ``Raise ValueError(10)`` -> `10`.
    * ``traceback`` -> The traceback ``instance``.
    * ``Note:`` If no unhandled exceptions occurred, all three are `None`, making them `Optional`.

If we had to document those in terms of python types, it would look something like this:

    .. code-block:: python

        from typing import Type
        from typing import Optional
        from types import TracebackType
        from contextlib import AbstractContextManager

        class K(AbstractContextManager):  # This implements a self returning __enter__ mixin.
            def __exit__(self,
                exc_type: Optional[Type[BaseException]],
                exc_value: Optional[BaseException],
                traceback: Optional[TracebackType]
            ):
                print(exc_type, exc_value, traceback)

        with K(): as k:
            ...
        # None, None, None -> No exception was raised and unhandled!

        with K() as k:
            try:
                raise ValueError(10)
            except ValueError:
                ...
        # None, None, None -> Raised exception was handled.

        with K() as k:
            raise ValueError(100)

        # <class `ValueError`>, 100, <traceback object at 0x7f052c53d200> (unhandled exception).

A word of warning about ``__exit__``, the return type of dunder exit is evaluated in a boolean context where
``truthy`` values result in suppressing unhandled exceptions.  Dunder ``__exit__`` should also avoid re raising
the exception which is passed in by python when unhandled exceptions occur in the runtime context, this is the
responsibility of the caller.


    .. code-block:: python

        from contextlib import AbstractContextManager

        class SuppressedExc(AbstractContextManager):
            def __exit__(self, exc_type, exc_value, traceback):
                return True  # Truthy -> True, suppresses exceptions...!


        with SuppressedExc() as s:
            raise ValueError(100)

        # No exception raised here!

        class NotSuppressedExc(AbstractContextManager):
            def __exit__(self, exc_type, exc_value, traceback):
                return False

        with NotSuppressedExc() as ns:
            raise ValueError(200)

        """
        ValueError                                Traceback (most recent call last)
        <ipython-input-7-55fb72d3f55a> in <module>
              1 with NotSuppressedExc() as ns:
        ----> 2     raise ValueError(200)
              3

        ValueError: 200
        """

Context Managers: contextlib
-----------------------------

Python ships out of the box with the ``contextlib`` module, which is a utility module for using
various python context managers as well as some context managers that can make using other non
context managers easier.

Context Managers: closing
--------------------------

The ``contextlib.closing`` context manager can be used to automatically close another object that
itself is maybe not necessarily a context manager.  It simply takes the object instance and calls
a `.close()` method on it, in a nutshell it would be like this:

    .. code-block:: python

        from contextlib import contextmanager

        @contextmanager
        def close_it(obj):
            try:
                yield obj
            finally:
                obj.close()

this allows us to write code like this for any object that has a `.close()` method but itself is
not a context manager.

    .. code-block:: python

        from urllib.request import urlopen

        with close_it(urlopen("https://www.google.com")) as page:
            for line in page:
                print(line)

Even if an exception is raised here, the `page` will always have `.close()` invoked on it.

Context Managers: nullcontext
------------------------------

``contextlib.nullcontext`` can be used to return a no-op, it is intended for use as a stand
in for an optional context manager.  Based on some logic, e.g some ``if`` clause, you may
use a ``nullcontext``, a good example of such a use case is:

    .. code-block:: python
    from contextlib import nullcontext
    from contextlib import suppress

    def function(ignore_exceptions: bool = False):
        mgr = suppress(Exception) if ignore_exceptions else nullcontext()
        with mgr:
            ...  # Do something, depending on the function arg, exceptions are suppressed!

Basically if you may want to run some sort of context manager or not based on some branched
logic in your code, ``nullcontext`` can be used as a standard in to fill the gap in some
alternative case.

Context Managers: suppress
---------------------------

Often it is necessary to run some piece of code while ignoring an assortment of exceptions, simplifying
a ``try: except: pass`` kind of setup.

    .. code-block:: python

        from contextlib import suppress

        # Approach 1
        def try_something():
            try:
                do_some_operation()
            except ValueError:
                pass


        def with_suppress():
            with suppress(ValueError):
                do_some_operation()

