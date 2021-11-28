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

Returns a new `bytes` object of the string.





