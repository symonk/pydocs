Python Built in Functions
==========================

Built ins: abs()
-----------------

`abs(x)` returns the `absolute` value of a number.  When we say `absolute` we mean,
how far from 0 the number is, be it positive or negative.  `abs()` accepts:

    * an integer.
    * a floating point number.
    * any object implementing dunder `__abs__`.
    * returns the `magnitude` for `Complex` number types.

    .. code-block:: python

        abs(100)  # 100
        abs(-100)  # 100
        abs(-125.75)  # 125.75
        abs(complex(5, 10))  # 11.180339887498949

        class Example:
            def __init__(self, x: int) -> None:
                self.x = x

            def __abs__(self) -> int:
                # Dummy implementation, double the real abs
                return abs(self.x) * 2

        abs(Example(100)) # 200