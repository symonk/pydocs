Pass By Assignment
===================

Pass By Assignment
-------------------
In python, arguments are *passed by assignment*.  Not to be confused with `pass by reference` or
`pass by value`.  The rationale behind this is two fold:

    - The parameter passed in by the caller is actually a `reference` to an object.
    - Some data types are immutable (e.g `int`, `string`, `bytes`, `tuple` etc..)

What this means in practice is:

    - If the caller passes a `mutable` object to a method, the method itself gets a reference
to that same object and it can be mutated as desired, however rebinding the reference inside the
method will **NOT** reflect to the outer scope (callers) reference.  Rebinding and mutating will
not reflect on the callers object.

    - If the caller passes a `mutable` object, rebinding the reference will also not impact the
callers object and you will not be able to update the state.


Pass By Assignment: Mutable
----------------------------

    .. code-block:: python


        mutable = [1,2,3]

        def demo_mutable(data):
            mutable.append(4)  # This will modify the outer scope, the callers `mutable` will be updated after.

        print(mutable)
        # [1,2,3,4]

        def demo_rebinding(data):
            # remember data[-1] == 4 now.
            data = [1,2,3,4,5,6,7]

        print(mutable)
        # [1, 2, 3, 4] - outer scope `mutable` is not impacted due to rebinding inside `demo_rebinding(...)`


Pass By Assignment: Immutable
------------------------------


    .. code-block:: python


        immutable = "string"

        def demo_mutable(data):
            data += "foo"

        print(immutable)
        # 'string'

        def demo_rebinding(data):
            data = "foo"

        print(immutable)
        # 'string'
