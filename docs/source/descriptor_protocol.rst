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

            def __get__(self, obj, objtype=None):
                print(locals())


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

