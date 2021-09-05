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
        pattern += "[A-Z]{1}"  # First character MUST be an uppercased A -> Z character
        pattern += "[a-zA-Z0-9]{8,}  # Must then contain AT LEAST 8 additional characters /1[8+]1/
        # We have implicitly guaranteed so far that we have an uppercase char[0] and 8 alpha numeric chars ending in a digit.
        pattern += "[0-9]{1}"  # Must end with a number

        # putting it altogether then, pattern is:
        pattern = r'[A-Z]{1}[a-zA-Z0-9]{8,}[0-9]{1}'
        re.match(pattern, "ValidPassword2")  # <reMatch object; spam=(0, 14), match='ValidPassword2'>
        re.match(pattern, "invalidPassword5") # None (no match due to missing initial capital)
        re.match(pattern, "InvalidPassword")  # None (no match due to missing ending digit
        re.match(pattern, "Invalid5")  # None, too short!

If you are experienced in regular expressions; you may be screaming that there are *other* or *better* ways to
do exactly this; often with regular expressions there are many ways to skin a cat, but for simplicity and to serve
as an introduction, this is a decent enough example.  Again, if this is all new to you, focus on trying to understand
it but not remember it, we will be going in-depth shortly.

**Note:** Here we are reusing the ``pattern`` string,  it is advisable when reusing a pattern to compile it into a
``re.Pattern`` object using ``re.compile(pattern)``.


Regular Expr: Simple Matchers
------------------------------

In it's simplest form, a regular expression is just a bunch of characters that we use to perform a search
in a string, for each snippet in this article we will be sharing an example of the syntax in action as well
as an interactive link to dabble and view it yourself.


    .. list-table:: Simple Matcher
        :header-rows: 1

        * - Pattern
          - Subject String
          - Expected Match
        * - example
          - This is a trivial example
          - This is a trivial **example**
        * - bar
          - Foo bar
          - Foo **bar**

Try Simple Matcher: https://regex101.com/r/xlfL9n/

Typically regular expressions are case **insensitive**, (outside of using the `i` flag - more on that towards
the end of the article under the `flags` section).

    .. code-block:: python

        import re
        re.match("foo", "Foo will not match")


Regular Expr: Meta Characters
------------------------------
Meta characters are the bread and butter of regular expressions, and understanding them can make staring at
a daunting regular expression become somewhat demystified.  Here is a brief summary of the core meta characters:


    .. list-table:: Regex Meta Characters
        :header-rows: 1

        * - Meta Characters
          - Description
        * - ``.``
          - Period matches any single character, except a line break character e.g `\n`
        * - ``[]``
          - Character classes.  Match any character contained within the brackets.
        * - ``[^]``
          - Negated Character classes.  Match any character **NOT** contained within the brackets.
        * - ``?``
          - Makes the preceding symbol *optional*.
        * - ``+``
          - Matches **one** or more of the preceding symbol.
        * - ``*``
          - Matches **zero**` or more of the preceding symbol.
        * - ``{i, j}``
          - Braces. Matches **at least** `i` but no more than `j` repetitions of the preceding symbol.
        * - ``(foo)``
          - Character group. Matches the characters `foo` in exactly that order.
        * - ``|``
          - Alternation.  Matches characters either before **or** after the symbol.
        * - ``\``
          - Escapes the next character, This allows using meta characters (and others) in their literal sense.
        * - ``^``
          - Carat. Matches the beginning of the input (also has use in negative character classes).
        * - ``$``
          - Dollar sign.  Matches the end of the input.  `^foo$`.


Regular Expr: Meta -> .
-----------------------
The meta character `.` is used to indicate any single character.  This has some exclusions for things like line breaks
and it is also worth noting that certain language re implementations can permit flags which also allow this character
to match even line breaks as well, we will discuss that here using pythons ``DOTALL`` flag.


    .. list-table:: Meta Full Stop
        :header-rows: 1

        * - Pattern
          - Subject String
          - Expected Match
        * - ``.at``
          - I put a hat on my cat
          - I put a **hat** on my **cat**
        * - ``foo.``
          - foo1 with foo2
          - **foo1** with **foo2**

Try Full Stop: https://regex101.com/r/AcAdBK/1


    .. code-block:: python

        import re
        pattern = r"foo."
        re.findall(pattern, "foo1 with foo2")
        # ["foo1", "foo2"]


Line breaks and pythons `DOTALL` flag example:

    .. code-block:: python

        import re
        foo = "foo\n"
        re.match("foo.", foo)
        #  No Match as `.` does not match on the new line
        re.match("foo.", foo, flags=re.DOTALL)  # Capture line breaks too!
        # < re.Match object; span=(0,4), match='foo\n'>


Regular Expr: Character Classes -> [...]
---------------------------------------
Character classes in regex are used to denote literal values, so using meta characters inside
them do not need escaped.  Hyphens can be used inside character classes to signify a range,
just like we used in the initial example (username validation).  Character classes are denoted
by the ``[`` <--> ``]`` square brackets.  Order inside character classes does **not** matter:

    .. list-table:: Meta Character Classes
        ..:header-rows: 1

        * - Pattern
          - Subject String
          - Expected Match
        * - ``[Tt]he .at``
          - The cat
          - **The cat**
        * - ``[sMc]at``
          - The cat, sat on the Mat
          - The Foobar, was **foobar**

Try Character Classes:  https://regex101.com/r/Dhw7Dt/1

    .. code-block:: python

        import re
        pattern = re.compile(r"[sMc]at")
        re.findall(pattern, "The cat sat on the Mat")
        # ['cat', 'sat', 'Mat']


Regular Expr: Negated Character Classes -> [^...]
---------------------------------------------------
Similar to the ``Character Classes`` outlined previously, the negated character class matches
anything **except** what is defined inside the square brackets.  We mentioned previously how
the carat ``^`` symbol can denote the start of the string, however it's additional use case
is here (as well as in `lookarounds` more on that one later..).  Here we will find any words
that do **NOT** start with a letter:

    .. list-table:: Meta Negated Character Classes
        ..:header-rows: 1

        * - Pattern
          - Subject String
          - Expected Match
        * - ``[^a-zA-Z]*``
          - NoMatch
          - <no match>
        * - ``[^a-zA-Z]*``
          - 5Matched
          - **5Matched**

Try Negated Character Classes:  https://regex101.com/r/r8rAuc/1

    .. code-block:: python

        import re

        pattern = re.compile(r"[^a-zA-Z]*")
        re.match(pattern, "failed")
        re.match(pattern, "5Passed")

**Note**:  There are some short hand tricks with regex, which we will discuss later, things like `\d` and `\w`
but for simplicity, bear with me for now.  You will also notice various methods of the python ``re`` module here,
the difference between ``re.search``, ``re.match`` and ``re.findall`` will be outlined later on as well.

Regular Expr: Question Mark -> ?
-------------------------------------
The meta character ``?`` indicates an **optional** preceding character (or group).  This matches
**zero** or more of the preceding character.

    .. list-table:: Meta Optional Repitition
        .. :header-rows: 1

        * - Pattern
          - Subject String
          - Expected Match
        * - ``[T|t]?he``
          - he
          - **he**
        * - ``[T|t]?he``
          - The
          - **The**

Try Optional Repitition: https://regex101.com/r/xWK4S7/1

    .. code-block:: python

        import re
        pattern = re.compile(r"[T|S]?he")
        re.match(pattern, "The")  # <re.Match object; span=(0, 3), match='The'>
        re.match(pattern, "She")  # <re.Match object; span=(0, 3), match='She'>
        re.match(pattern, "he")  # <re.Match object; span=(0, 2), match='he'>

Regular Expr: Plus -> +
-----------------------------------
