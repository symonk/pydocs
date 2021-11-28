String Methods
================


capitalize
------------

Returns a copy of the string with s[0] capitalized and the rest (s[1:]) lowercased.  Capitalize(...) accepts no arguments.

  .. code-block:: python
  
    s = "hello world."
    print(s.capitalize()) # `Hello world.`
    

casefold
---------

Returns a copy of the string with a stricter lower case enforcing; by default `lower()` does not account for various code
points, such as `ß`, using `casefold()` is more aggressive and will convert such characters to `ss`.  `casefold()` takes
no arguments.

  .. code-block:: python
    
    s = "foo ß"
    print(s.lower()) # `foo ß`
    print(s.casefold()) # `foo ss`
    
    
center
-------

Returns a copy of the string centered with a length of `width`, padded by a `fillchar`.  The argument width is
required, fillchar is optional and by default is the ASCII space (code point: `U+0020`).  By default no keyword args
are supported, width and fillchar are positional only. In the event that `width` is less than the `len(s)` then the
original `s` string is returned.

    .. code-block:: python

        s = "center me"
        print(s.center(20, "#"))  # `#####center me######`
        x = "example"
        print(s.center(30))  # `           example            `

        # short width
        s = "too short"
        print(s.center(3)) # `too short`


count
------

Returns the number of non overlapping occurrences of `sub` in the string.  `count(sub, [,start[, end]])`
can accept an optional `start` and `end` argument to perform the check on a particular range, interpreted
in `slice` notation e.g `s[start:end]`.  `count(...)` accepts no keyword and only positional arguments.


    .. code-block:: python

        s = "example of examples"
        target = "example"
        print(s.count(target))  # 2
        print(s.count(target, 0, len(target)) # 1

        # Overlapping example
        s = "ababab"
        print(s.count("ab")) # 3 ( non overlapping)
        print(s.count("aba")) # 1 (You may think, `2` but infact it's `1`)


encode
-------

Returns a new `bytes` version of the encoded string.  As of recently `encode(encoding=..., errors=...)` via
keyword arguments is supported.  By default encoding is `utf-8` being the most prevalent in present times
and errors is `strict` however additional options are: (`ignore`, `replace`, `xmlcharrefreplace` and
`backslashreplace`).  User defined additional ones can be provided if they are registered via the codec module
using `codecs.register_error(...)`.  By default encoding errors raise a `UnicodeError`.  Note: Python as a
language is unicode aware, this is demonstrated in the examples below:

    .. code-block:: python

        s = "foo 😊 bar"
        print(type(s.encode()) # `bytes`
        print(s.encode()) # b'foo \xf0\x9f\x98\x8a bar'
        print(len(s.encode())) # 12 bytes
        # `foo` as normal (3 bytes) + suffix whitespace (1 byte) (4)
        # Emoji 😊 is 4 byte UTF-8 (U+1F60A) (4 bytes)
        # `bar as normal` (3 bytes) + prefix whitespace (1 byte) (4)
        # 3 + 1 + 4 + 1 + 3 (12 bytes).

        # Keyword args are supported.
        s = "foobar"
        print(s.encode(encoding="utf-8", errors="strict")) # b'foobar' (6 bytes).


endswith
---------

Check if a string is suffixed with a particular substring.  Optional `start` and `end` arguments can be
provided which again are interpreted in `slice` notation.  The suffix parameter can also be a tuple of
various suffixes to look for.  `endswith(...)` does not support keyword arguments, positional only.

    .. code-block:: python

        s = "language:html"
        print(s.endswith(("html", "php"))  # True
        print(s.endswith("guage", 0, 8))  # True


expandtabs
-----------




