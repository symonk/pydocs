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
required, fillchar is optional and by default is the ASCII space (code point: ``).  By default no keyword args
are supported, width and fillchar are positional only.

    .. code-block:: python

        s = "center me"
        print(s.center(20, "#"))  # `#####center me######`
        x = "example"
        print(s.center(30))  # `           example            `


