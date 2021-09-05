Regular Expressions
====================

A regular expression is a pattern that is matched against a subject string, from left to right.
Some common uses of regular expressions (but not exhaustive) are:

    * Replacing text within a string
    * Capturing groups of information from a string
    * Validating data, like a user name with multiple constraints
    * much more...


A Trivial Example
------------------
As we previously just mentioned, a common use case for regex is validating user input against
an assortment of different constraints.  To take the user name validation example, let's look
at how we might validate the following constraints on user input:

    * Must begin with a capital letter
    * Must be at least 10 characters in length
    * Must end with a number
    * Can only be alphanumeric

This is a simple example, and the point is just to serve as an introduction to regular expressions.
Let's have a look at how we would implement such a scenario in python and explain step by step
what each part is doing.  Don't worry too much about understanding to following as in this article
we will be breaking down the core fundamentals of regular expressions into digestable chunks.  The
aim by the end of it, is that you should be able to piece together complex expressions for matching
an assortment of scenarios:


    .. code-block:: python

        import re  # Import pythons regular expressions module

        # For demonstration purposes; we will build the string over multiple steps
        # for ease of understanding
        pattern = r"" # starting point; empty raw string
        pattern += "[A-Z]{1}"  # First character MUST be an uppercased a...z character
        pattern += "[a-zA-Z0-9]{9,}  # Must then contain AT LEAST 9 additional characters (our 10+ constraint counting our uppercase)
        # We have implicitly guaranteed so far that we have an uppercase char[0] and 9 alpha numeric chars atleast
        pattern += "[0-9]{1}"  # Must end with a number

        # putting it altogether then, pattern is:
        pattern = r'[A-Z]{1}[a-zA-Z0-9]{9,}[0-9]{1}'
        re.match(pattern, "ValidPassword2")  # <reMatch object; spam=(0, 14), match='ValidPassword2'>
        re.match(pattern, "invalidPassword5") # None (no match due to missing initial capital)
        re.match(pattern, "InvalidPassword")  # None (no match due to missing ending digit
        re.match(pattern, "Invalid5")  # None, too short!

If you are experienced in regular expressions; you may be screaming that there are *other* or *better* ways to
do exactly this; often with regular expressions there are many ways to skin a cat, but for simplicity and to serve
as an introduction, this is a decent enough example.  Again, if this is all new to you, focus on trying to understand
it but not remember it, we will be going in-depth shortly.

**Note:** Here we are reusing the `pattern` string,  it is advisable when reusing a pattern to compile it into a
``re.Pattern`` object using ``re.compile(pattern)``.


Regular Expr: Simple Matchers
------------------------------

In it's simplest form, a regular expression is just a bunch of characters that we use to perform a search
in a string, for each snippet in this article we will be sharing an example of the syntax in action as well
as an interactive link to dabble and view it yourself.

    .. list-table:: Simple Matcher
        :header-rows: 1

        * - Pattern, Subject String, Expected Match
        * - example, This is a trivial example, This is a trivial **example**
          - bar, Foo bar, Foo **bar**

Try Simple Matcher: https://regex101.com/r/xlfL9n/



