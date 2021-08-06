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

            def __get__(self, obj, objtype = None):
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
            def __get__(self, obj, objtype=None):
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
            def __get__(self, obj, objtype = None):
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
            def __get__(self, obj, objtype=None):
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

Descriptors: Customising names
-------------------------------

When a class uses ``descriptors``, it can inform the descriptor of which variable
name was used, this can help us circumvent the issue we exposed during our managed
attribute example.  This is achieved through the dunder `__set_name__` method,
below is an example where multiple variables can become managed attributes without
lots of coupling in the Descriptor implementation itself:

    .. code-block:: python

        import logging
        logging.basicConfig(level=logging.INFO)  # root logger configured to info

        class LoggedAttr:
            def __set_name__(self, owner, name):
                # This is new! it holds the key to decoupling multiple managed attributes
                # Let's store a public/private names on the actual Descriptor instance
                logging.info("__set_name__ called!", locals())
                self.public = name
                self.private = "_" + name

            def __get__(self, obj, objtype = None):
                private = getattr(obj, self.private)
                logging.info(f"Retrieving: {self.public} with value: {private}")
                return private

            def __set__(self, obj, value) -> None:
                logging.info(f"Updating: {self.public} to: {value}")
                setattr(obj, self.private, value)

        class Car:
            wheels = LoggedAttr()
            color = LoggedAttr()

            def __init__(self, wheels, color):
                self.wheels = wheels
                self.color = color

            def remodel(self):
                self.wheels = 3
                self.color = "blue"

        c = Car(4, "red")
        # INFO:root:Updating: wheels to: 4
        # INFO:root:Updating: color to: red
        c.remodel()
        # INFO:root:Updating: wheels to: 3
        # INFO:root:Updating: color to: blue

As you can see, the same ``LoggedAttr`` class is now capable of supporting multiple attributes, all
handled by the magic of `__set_name__` which aids in setting up attribute name specific values for
public and private in the `LoggedAttr` instance namespace.  The important thing to understand here
is that `LoggedAttr` instances are invoked at the class level, during interpretation of the ``Car``
class, before a ``Car`` instance has been created in memory, the ``__set_name__`` was already
invoked, twice by python.  Let's now understand ``__set_name__`` a little better.

Descriptors: __set_name__
--------------------------

Dunder `__set_name__` is called when the descriptors owning class is **created**.  Note: This is **not** to be
confused with instantiating an instance of the owner class, remember classes themselves are objects in python.

A very important fact of the ``__set_name__`` dunder is that it is only called as part of the ``type`` constructor.
(to understand more about ``type``, refer to my article on ``metaclassess` in python3).  This means that if a
descriptor is dynamically bolted on after the fact, ``__set_name__`` would need to be explicitly called.  This
is outlined below:

    .. code-block:: python

        class Klazz:
            ...

        descriptor = MyDescriptor()
        Klazz.d = descriptor  # This is not sufficient.
        descriptor.__set_name__(Klazz)  # Retrospectively, explicitly call __set_name__.


Descriptors: __delete__
------------------------

The final piece of the descriptor protocol, `__delete__()` is called to delete an attribute
on an instance of the owner class.  Implementing a ``__delete__()`` is enough to qualify
the descriptor as a ``Data Descriptor``.  This is outlined below:

    .. code-block:: python

        class D:

            def __delete__(self, obj):
                # self -> the instance of D
                # obj -> the instance of the owner class (where D() was instantiated at the class level)
                print("deleting x")

        class S:
            d = D()

        s = S()
        del s.d
        # deleting x


Descriptors: Summary
---------------------

    *  A ``descriptor`` is any object that implements:
        * `__get__`, `__set__`, `__delete__`
    * Optionally, descriptors can have a `__set_name__` if they need to know:
        * The ``class`` they where created.
        * The name of the variable they where assigned too.
    * ``__set_name__`` is invoked even for classes which are **not** descriptors.
    * Descriptors get invoked by the `dot` operator, during attribute lookup.
    * Accessing a descriptor indirectly, the descriptor instance is not invoked but returned:
        * vars(Klazz)['descriptor']  # returns the descriptor instance, but does not invoke __get__() etc.
        * ``Klazz().__class__.x`` != ``Klazz().__class__.__dict__['x']``.
    * Descriptors only work as ``class`` variables, stored in an instance has no effect.
    * The main motivation for descriptors is to allow ``class level`` attributes to have a hook into attribute access.
    * In a normal setup, the ``calling`` class controls what happens during lookup.
    * Descriptors invert the control and allow the data being accessed to have a say in the matter.


Descriptors: A Real use case
-----------------------------

So far, we have developed relatively trivial uses for python descriptors.  Now we will
put together all we have learned to implement a real use case.  In this example we
will build a `Field` descriptor that can validate data inputs, we will create a ``BoundedInteger``
to validate integers in a reusable, strict manner:

    .. code-block:: python

        from abc import ABC
        from abc import abstractmethod


        class Field:
            def __set_name__(self, owner, name):
                self.private_name = "_" + name

            def __get__(self, obj, objtype = None):
                return getattr(obj, self.private_name)

            def __set__(self, obj, value):
                self.validate(value)
                setattr(obj, self.private_name, value)  # noqa

            @abstractmethod
            def validate(self, value):
                ...


        class BoundedInteger(Field):

            def __init__(self, min: int = 0, max: int = 256):
                self.min = min
                self.max = max

            def validate(self, value):
                # For the sake of this demo, we want fine grained error messages!
                if not isinstance(value, int):
                    raise TypeError(f"Expected {value!r} to be an integer")
                if not isinstance(self.min, int):
                    raise TypeError(f"Expected {self.min} to be an integer")
                if not isinstance(self.max, int):
                    raise TypeError(f"Expected {self.max} to be an integer")
                if not self.min <= value <= self.max:
                    raise ValueError(f"{value} was not between: {self.min}, {self.max} [inclusive]")


        class RequiresValidation:
            value = BoundedInteger(min=0, max=10)

            def __init__(self, value):
                self.value = value

        # Let's try it out!
        RequiresValidation("foo")
        """
        Traceback (most recent call last):
          File "validation.py", line 47, in <module>
            RequiresValidation("foo")
          File "validation.py", line 43, in __init__
            self.value = value
          File "validation.py", line 13, in __set__
            self.validate(value)
          File "validation.py", line 30, in validate
            raise TypeError(f"Expected {value!r} to be an integer")
        TypeError: Expected 'foo' to be an integer
        """

        RequiresValidation(7.5)
        """
        Traceback (most recent call last):
          File "validation.py", line 47, in <module>
            RequiresValidation(7.5)
          File "validation.py", line 43, in __init__
            self.value = value
          File "validation.py", line 13, in __set__
            self.validate(value)
          File "validation.py", line 30, in validate
            raise TypeError(f"Expected {value!r} to be an integer")
        TypeError: Expected 7.5 to be an integer
        """

        RequiresValidation(-1)
        """
        Traceback (most recent call last):
          File "validation.py", line 47, in <module>
            RequiresValidation(-1)
          File "validation.py", line 43, in __init__
            self.value = value
          File "validation.py", line 13, in __set__
            self.validate(value)
          File "validation.py", line 36, in validate
            raise ValueError(f"{value} was not between: {self.min}, {self.max} [inclusive]")
        ValueError: -1 was not between: 0, 10 [inclusive]
        """

        RequiresValidation(11)
        """
        Traceback (most recent call last):
          File "validation.py", line 47, in <module>
            RequiresValidation(11)
          File "validation.py", line 43, in __init__
            self.value = value
          File "validation.py", line 13, in __set__
            self.validate(value)
          File "validation.py", line 36, in validate
            raise ValueError(f"{value} was not between: {self.min}, {self.max} [inclusive]")
        ValueError: 11 was not between: 0, 10 [inclusive]
            """

Descriptors: Advanced
----------------------

Up until now we have skimmed the technical internals of descriptors.  It is important to grasp
the previous concepts well before looking any deeper into the attribute lookup call flow etc.

We briefly touched on ``data`` and ``non data`` descriptors and mentioned how depending on which
one the descriptor implementation is 'classified' as, has impacts on the attribute lookup
call flow.  To recap:

    * Descriptor protocol consists of `__get__`, `__set__` and `__delete__`.
    * Implementing any of the above qualifies.
    * If only `__get__` is implemented, it is known as a ``Non Data`` descriptor
    * If ``__get__` + `__set__` || `__delete__` are implemented, it is known as a ``Data`` descriptor.

The default behaviour for attribute access is to get, set or delete an attribute from an object dictionary.
for example:

    * Firstly object_instance.attribute firstly looks for `attribute` in `object_instance.__dict__`
    * Secondly, ``type(object_instance).__dict__``
    * Thirdly, resolving the ``mro`` of ``type(object_instance)``.
    * If the looked up object is a descriptor, python may invoke the descriptor instead
    * note: Depending on which descriptor protocols are implemented, mileage varies.

Descriptors: The Protocol
--------------------------


