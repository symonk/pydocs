Python Object copying
======================

Copying: Shallow Copy
----------------------

Shallow copying a container in python builds a new instance of the container type, but the elements
inside the container are just references to the same objects in the container prior to copying.  This means
that if you shallow copy a list for example, and update the index[0], both lists will be updated as they are
the same object in memory, let's try it out:

    .. code-block:: python

        items = [1,2,3,4,5]
        new_items = items.copy()
        items[0] = 10
        items [10, 2, ...]
        new_items  # [10, 2, ...]


Copying: Deep Copy
-------------------

...