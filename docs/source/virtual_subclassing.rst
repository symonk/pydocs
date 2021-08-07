Virtual Subclassing
====================

Python Virtual subclassing
---------------------------

Unlike some other languages, python supports the concept of ``virtual subclassing``.  What exactly
is virtual subclassing?  To understand the concept let's go back to pythons roots.  I am sure
you are familiar with the concept of `duck typing` and perhaps if you have came from another
language such as ``java``, you will be familiar with the concept of an ``interface``.  In python
we are not overly caught up on the `type` of `X`, but more so how `X` behaves, in the simplest
form if it walks like a duck and quacks like a duck, chances are its a duck.  Python does not support
an official ``interface`` argument, however more recently ``Protocols`` can be ``@runtime_checkable``
to aid with catching issues early.

To understand why we may need virtual subclassing, let's take a simple example.  Firstly we are tasked
with developing an airplane tycoon game.  Our game consists of multiple things that can fly, to use
the age old cliche:

    * A Bird
    * An Aeroplane

Turns out, you cannot load people into A Bird and fly them to a new destination, so we develop a few
abstract base classes to easily distinguish them

    .. code-block:: python

        from abc import ABC
        from abc import abstractmethod

        class Plane(ABC):
            @abstractmethod
            def fly(self):
                ...

        # We make a simple plane

        class MyPlane(Plane):
            def fly(self):
                print("Plane preparing to fly!")


        # Now we can check in our library function
        # Note: isinstance, isinstance checks against an ABC are even questionable..
        def generate_flight(plane: Plane):
            if not isinstance(plane, Plane):
                raise TypeError("Not a Plane!")
            plane.fly()

        # Now, if we accidentally receive a Bird, we will handle the case
        # This will allow us to handle it gracefully rather than potentially
        # Blow up with some other weird errors later on in the program
        class Bird:
            def fly():
                print("bird flying!")

        generate_flight(Bird())
        """
            TypeError                                 Traceback (most recent call last)
            <ipython-input-16-be9dc02ec41e> in <module>
            ----> 1 generate_flight(Bird())

            <ipython-input-15-d59f577c3fde> in generate_flight(plane)
                 18 def generate_flight(plane: MyPlane):
                 19     if not isinstance(plane, MyPlane):
            ---> 20         raise TypeError("Not a Plane!")
                 21     plane.fly()
                 22

            TypeError: Not a Plane!
        """
        generate_flight(MyPlane())
        # Plane preparing to fly!

Excellent, but whats the point in virtual subclassing still?  Let's say in the near future some other
fantastic library comes along with a dozen of cool new planes, the problem is our ``generate_flight()``
function here is strictly prohibiting non explicit subclasses of ``Plane``.  Let's take a look at
the library code

    .. code-block:: python

        # snippet from another library, not developed by you
        # boeing.py
        class Boeing747:
            def fly(self):
                print("Boeing 747 preparing for travel!")

            def repair(self):
                ...

Pretty simple huh, a nice shiny new Boeing that we could use in our system, except of course it does
not adhere to our explicit abstract base class:

    .. code-block:: python

        from boeing import Boeing747

        generate_flight(Boeing747())
        """
        TypeError                                 Traceback (most recent call last)
        <ipython-input-23-24437f4c4918> in <module>
        ----> 1 generate_flight(Boeing747())

        <ipython-input-20-8038f0273ec8> in generate_flight(plane)
             18 def generate_flight(plane: MyPlane):
             19     if not isinstance(plane, MyPlane):
        ---> 20         raise TypeError("Not a Plane!")
             21     plane.fly()
             22

        TypeError: Not a Plane!
        """

Damn, we have coupled our library a little too tight and we don't own the library code, what gives?
Rather than monkey patching and hacking around the inheritance of ``Boeing747``, enter ``virtual subclassing``.
We can simple register the third party code as a virtual subclass of our _interface_ (abstract base class) and
the python interpreter will treat it like it has actually subclassed it.

    .. code-block:: python

        from boeing import Boeing747

        Plane.register(Boeing747)
        isinstance(Boeing747(), Plane)  # True!
        issubclass(Boeing747, Plane)  # Also True!
        generate_flight(Boeing747())
        # Boeing 747 preparing for travel!

Voila, we have successfully used third party code and ackknowledged explicitly that we accept it is an
adaquate implementation of our interface.

Note: virtual subclassing should be used extremely sparingly, in reality the need for it is often miniscule.
It is also possible to automatically consider objects as instance/subclasses based on their interface and
python does this internally a lot in its collections.abc module, more on that in a separate post alter.