Positional and Keyword arguments
=================================

Function arguments: Introduction
---------------------------------
Python offers three main ways to control function arguments, these are:
    * positional only
    * positional or keyword
    * keyword only

Here is a short overview of what we will cover here:

    .. code-block:: python

        def example(pos_only /, pos_or_kwd, *, kwd_only)
            # pos_only is positional only, order matters and cannot be passed via pos_only=1
            # pos_or_kwd like traditional args (without /, *) can be specified by order (pos) or keyword explicitly
            # kwd_only must be specified as kwd_only=10
            ...

        example(1, 2, kwd_only=3)  # Valid
        example(1, pos_or_kwd=2, kwd_only=3)  # Valid
        example(1,2,3) # Invalid (kwd_only must be keyword only)
        example(pos_only=1, 2, kwd_only=3)  # Invalid (pos_only cannot be specified via keyword)

Positional Only Arguments
--------------------------

Arguments specified before a `/` in a function signature, indicate the argument should only be
passed as a positional argument (e.g without a keyword and order matters):

    .. code-block:: python

        def one(a, b, c, /):
            ...

In the above example, a,b,c must be passed explicitly without a keyword argument.

Positional OR Keyword Arguments
--------------------------------
By default, when either `/` or `*` is omitted from a function signature, arguments are
considered `positional_or_keyword` args, which means passing them positionally in order
is completely valid and passing them via keyword explicitly is also valid:

    .. code-block:: python

        def one(a,b,c):
            ...

        one(1,2,3)  # Valid
        one(a=10, b=20, c=30)  # Valid

Keyword only Arguments
-----------------------
