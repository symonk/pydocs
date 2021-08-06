Pythons Descriptor Protocol
===========================

Descriptors Summmary
---------------------
Python descriptors allow objects to customise:

    - Attribute lookup
    - Attribute storage
    - Attribute deletion

If you are new to descriptors, chances are you've already been using them
as they are responsible for underpinning core python built in functionality.
Some things which are powered by descriptors and we will discuss later are:

    - `@property`
    - `@staticmethod`
    - `@classmethod`
    - Creating `bound` methods from `function` types
    - Pythons `super()`
    - And much more...