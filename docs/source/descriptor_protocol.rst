Pythons Descriptor Protocol
===========================

Descriptors Summary
---------------------
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

Descriptors: Trivial Example
-----------------------------

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

Descriptors: Something more dynamic
------------------------------------



