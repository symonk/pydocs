Python Descriptor Protocol
===========================

Descriptors: Intro
----------------------------
Python descriptors allow objects to customise:

    * Attribute lookup
    * Attribute storage
    * Attribute deletion

If you are new to descriptors, chances are you've already been using them
as they are responsible for underpinning core python built in functionality.
Some things which are powered by descriptors and we will discuss later are:

    * `@property`
    * `@staticmethod`
    * `@classmethod`
    * Creating `bound` methods from `function` types
    * Pythons `super()`

Descriptors: A Trivial Example
-------------------------------

To get a feel for descriptors, we will create a rather trivial example, a
descriptor that reverses a simple string value, which importantly is computed
each time the value is accessed.  The important thing to understand at this
point are ``descriptors`` are their own class and they live as ``class``
attributes in other classes, let's see it action:

    .. code-block:: python


        class UpperAccess:
            """
            This is a simple descriptor, at this point we are only
            dealing with non data descriptors, so we will only
            implement __get__.  More on this later, keep reading!
            but let's keep things simple for now.
            """
            def __init__(self, word: str) -> None:
                self.word = word

            def __get__(self, obj, objtype = None) -> value:
                return self.word.upper()

        class UsingUpperAccess:
            """
            This is a simple class that uses a ``MyDescriptor`` instance.
            Important note: The descriptor must live as a CLASS attribute
            in another class.
            """
            word = UpperAccess("foo")

        clazz = UsingUpperAccess()
        print(clazz.word) # 'FOO'

The important thing to remember here is, ``descriptors`` live as class attributes
in other classes, when accessing the ``clazz.word`` python first has a look
in the `clazz.__dict__` <instance dict> and then finds the descriptor in the `type(clazz).__dict__` <class dict>.
The uppercased value does ``NOT`` live in either the `instance` or `class` dict, it is computed on demand!

Descriptors: Compute on demand
-------------------------------

To better explain the concept of value(s) being computed on demand, we will build a
descriptor instance that based on a working directory, can list the contents.  For
the sake of this article, we will use a `tree` structure like so:

    ::

        example
        ├── colors
        │   ├── blue.txt
        │   └── red.txt
        └── numbers
            ├── one.txt
            └── two.txt

In a nutshell, we have two subdirectories, `colors` and `numbers`, let's write a descriptor
that can read the contents of those files, dynamically:

    .. code-block:: python

        import os

        class ContentsOf:
            def __get__(self, obj, objtype=None) -> value:
                # it is the obj reference as a way back to the declaring class
                return os.listdir(obj.dirname)

        class RootDirectory:
            files = ContentsOf()

            def __init__(self, dirname):
                self.dirname = "/tmp/example/" + dirname

        colors = RootDirectory("colors")
        numbers = RootDirectory("numbers")
        colors.files # ["red.txt", "blue.txt"]
        numbers.files  # ["one.txt", "two.txt"]

Now that we understand a little better, how descriptors compute value(s) on demand, this
example also exposes us to a slightly deeper look into part of the ``descriptor protocol`.

Descriptors: __get__
-----------------------------

Part of the ``descriptor protocol``, dunder ``__get__`` is responsible for handling the
_lookup_ part of the descriptor outlined in our first paragraph.  The secret to understanding
how ``__get__`` works is to understand this is ``class level access``.

    .. code-block:: python

        class Descriptor:
            def __get__(self, obj, objtype = None) -> value:
                """
                :param self:
                    This instance of ``Descriptor``.

                :param obj:
                    The instance of the class in which the descriptor was instantiated

                :param objtype:
                    The (optional) own class `type` e.g `obj.__class__`

                __get__() should return the ``computed`` value, or raise an ``AttributeError``
                """
                ...

By default pythons `__get_attribute__` will provide both arguments to the `__get__` call, here is an
example of the types and value(s) accessible via `__get__()`:

    .. code-block:: python

        class D:

            def __get__(self, obj, objtype=None) -> value
                print(locals())
                # should really return here :)


        class Instance:
            d = D()

        i = Instance()
        i.d
        # {'self': <__main__.D object at 0x7f489e6f8340>,
        # 'obj': <__main__.Instance object at 0x7f489e8078b0>,
        # 'objtype': <class '__main__.Instance'>}
        # self -> the instance of `D`
        # obj -> the instance of `Instance`
        # objtype -> the class of instance `i.__class__`)`


Descriptors: Managed Attributes
--------------------------------

As we touched on originally in the form of pythons built in `@property`, a great example
use case for descriptors is managing access to instance data.  The descriptor is assigned
to a public attribute in the ``class`` dictionary (again not the actual value, it's computed
on demand) and the actual data is stored as a private attribute in the ``instance`` dictionary.
descriptors `__get__()` and `__set__()` are called for public access.  Up until now we have
only covered the `__get__()` part of the protocol, let's dive into what are known as
`Data Descriptors` (those which do not **only** implement `__get__()`, the former are known as
`Non Data Descriptors`.  We will create a guarded variable that when accessed audits its
access through python logging:


    .. code-block:: python

        import logging
        import random
        logging.basicConfig(level=logging.INFO)  # Simple root logger to info

        class LoggedAccess:
            def __get__(self, obj, objtype=None) -> value:
                private = obj._secure
                logging.info(f"Accessed `secure`, resulted in: {private}")
                return private

            def __set__(self, obj, value) -> None:
                # This is new to us, more on that after!
                logging.info(f"Setting `secure` to: {value}")
                obj._secure = value

        class Klazz:
            secure = LoggedAccess()  # Class dictionary, public attribute

            def __init__(self, secure):
                self.secure = secure

            def shuffle_secure(self):
                # shuffles the letters in our secure word!
                # Importantly, calls both __get__ & __set__ of our descriptor.
                new = list(self.secure)
                random.shuffle(new)
                self.secure = "".join(new)

        k = Klazz("nice")
        # INFO:root:Setting `secure` to: nice
        k.shuffle_secure()
        # INFO:root:Accessed `secure`, resulted in: nice
        # INFO:root:Setting `secure` to: inec

Looking closer at our example, we have derive a few things:

    * All access to the managed access `secure` is logged
    * `k` instance dictionary only holds the `_secure` attribute: `vars(k) -> {'_secure': 'inec'}`
    * `Klazz` class dictionary holds a instance of `LoggedAccess`: `vars(Klazz) -> `..., 'secure', ...`

One glaring problem with this is that our `_secure` attribute is hardwired and tightly coupled into the
`LoggedAccess` descriptor, this creates a bottleneck where each instance can only have a single logged
/ managed attribute and the name is completely unchangable.  We will discuss a solution to that later
but for now, let's understand the second piece of the descriptor procotol, `__set__`.

Descriptors: __set__
---------------------

Part of the ``descriptor protocol``, dunder ``__set__`` is responsible for handling the `storage`.
Descriptors implementing `__set__()` are automatically considered `Data Descriptors` and that
implicitly changes some of the attribute access flow, we will discuss that later.  Even if a
__set__ implementation has an exception raising place holder, it is enough to qualify as a
``Data Descriptor``.

    .. code-block:: python

        class Descriptor:
            def __set__(self, obj, value) -> None:
                """
                Called to update an attribute on the instance of the owner class
                Note: Adding a __set__() to a descriptor transforms it into a data descriptor
                which has impacts in terms of the call flow, more on that later.

                In typical setter fashion, __set__ should return `None`.
                """
                ...

Part of the ``descriptor protocol``, dunder ``__get__`` is responsible for handling the
_lookup_ part of the descriptor outlined in our first paragraph.  The secret to understanding
how ``__get__`` works is to understand this is ``class level access``.