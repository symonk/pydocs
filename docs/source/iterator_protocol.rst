Iterator Protocol
=========================

Python Iterator Protocol
-------------------------

Here we out cover the iterator protocol in depth, both the newer version via
``__iter__`` and ``__next__`` as well as the older protocol piggy backing
off ``__getitem__` for sequence types.

In python if you have found yourself wondering, how does the `for each` loop
work?  This is where the iterator protocol comes into play and making your
own iterable user defined types is surprisingly straight forward.  Personally
the terminology is more complex than the actual logic involved.  Let's
try and break it down:

    * ``collections.abc.Iterator`` extends ``collections.abc.Iterable``
    * All ``iterators`` are ``iterable``
    * Not all ``iterables`` are ``iterators``

Iterator Protocol: Iterable ABC
--------------------------------

``collections.abc.Iterable`` is an abstract base class built into python
that offers up the abstract ``__iter__`` method that should be implemented.
Rule of thumb is that anything that is ``iterable`` when asked for an
``iterator`` will return one.

    .. code-block:: python

        @abstractmethod
        def __iter__():
            while False:
                yield None

That's it, iterables are really that simple, if something is iterable, it can
return an ``iterator``.  It is an ``iterator` that python actually uses to
perform iteration.  The built in ``iter()`` function calls an objects
dunder ``__iter__``.

Iterator Protocol: Iterator ABC
--------------------------------

Once you have grasped that iterables are _responsible_ for returning ``iterators``,
things start to make a lot more sense.  Next up is the ``collections.abc.Iterator``
abstract base class provided by python and there are three main core things of note

    * Iterator extends Iterable.
    * Iterator implements a simple ``__iter__`` to make Iterable happy, that returns itself.
    * Iterator exposes a new ``__next__`` abstract method.

As we can see from inspecting the mro of ``collections.abc.Iterator``:

    .. code-block:: python

        from collections.abc import Iterator

        Iterator.__mro__
        #  collections.abc.Iterator, collections.abc.Iterable, object)


In order to satisfy the interface from ``collections.abc.Iterable``, it
implements a very basic ``__iter__``, which returns self:

    .. code-block:: python

        def __iter__():
            return self

However, iterators themselves offer up a little extra, the interface exposes
a new abstract method, known as ``__next__`` and it is this implementation
that when iterated over (for loops, `map`, list comps etc) that is used to
exhaust the iterator, one element at a time:

    .. code-block:: python

        def __next__(self):
            raise StopIteration

Something of note here, is unlike other languages, occasionally python uses
exceptions to handle code logic / flow, when an Iterator is exhausted it
should raise a ``StopIteration`` Exception, this is how python knows internally that
there are no more values.


Iterator Protocol: __getitem__
--------------------------------

There is another caveat, an object does not have to define the modern iterable/iterator
interfaces to qualify as being iterable, using dunder ``__getitem__`` if an object
can take an integer starting from `0`, python will happily iterate over that object as
well, this is known as the older iterator protocol, however due to the symantics,
when iterating using this approach, an ``IndexError`` should be raised instead of the
traditional ``StopIteration``.  Let's demonstrated an example:

    .. code-block:: python

        class ReversedEvenNumbers:
            def __init__(self, max):
                self.nums = [n for n in range(1, max+1)[::-1] if n % 2 == 0]

            def __getitem__(self, index):
                return self.nums[index]

        for n in ReversedEvenNumbers(15):
            print(n)
        # 14, 12, 10, 8, 6, 4, 2


As you can see, we have created something we can iterate over, without actually implementing
any of the iterator (modern) protocol.  Accessing an index out of range by default raises an
``IndexError`` so python gracefully handles that in this scenario.

Iterator Protocol: Modern Example
-----------------------------------

We have learned a little bit about the older iterator protocol with an example, however
let's implement something a little more modern.  Now we will use the abstract base
classes and create our own custom iterator and explain some of the magic behind
pythons virtual subclassing via ``abc.register`` and the ``__subclasshook__``.

In this example, we will be creating a word iterator from a user provided sentence.  Continue reading
after this topic to understand why our ``Sentence`` class does not have to explicitly inherit from
``collections.abc.Iterator`` (a little sprinkle of python magic!):

    .. code-block:: python

        # A typical first approach (albeit naive)
        class Sentence:
            def __init__(self, sentence: str) -> None:
                self.word_list = sentence.split()
                self.index = 0

            def __iter__(self):
                return self

            def __next__(self):
                if self.index >= len(self.word_list):
                    raise StopIteration
                value = self.word_list[self.index]
                self.index += 1
                return value


When starting out, you might think something like this is, pretty good.  However
there are a couple of caveats you should be aware of, each time iter(iterable)
is called, it should return a fresh ``iterator``.  What happens in this scenario
with the above implementation:

    .. code-block:: python
        s = Sentence("how are you")
        for w in s:
            print(w)
            # how
            # are
            # you
        for w in s:
            print(w)  # woops? we are already exhausted from previous!
            # <no output>

We need to create a fresh iterator, each time python calls ``__iter__`` on our object.  Let's patch
that up first:

    .. code-block:: python
        class RevisedSentence:
            def __init__(self, sentence: str):
                self.word_list = sentence.split()
                self.index = 0

            def __iter__(self):
                return iter(self.word_list)

            def __next__(self):
                if self.index >= len(self.word_list):
                    raise StopIteration
                value = self.word_list[self.index]
                self.index += 1
                return value

        rs = RevisedSentence("how are you")
        for w in s: print(w)
        # how
        # are
        # you
        for w in s: print(w)
        # how
        # are
        # you

Better! Each time we ask for an iterator from our custom ``RevisedSentence`` class, we
can access all the values, but can it be improved any more?  We'll, python supports
a ton of built iterators / iterables, we can much easier piggy back off those in
this kind of scenario:

    .. code-block:: python

        class SuperSentence:
            def __init__(self, sentence: str):
                self.word_list = sentence.split()

            def __iter__(self):
                return iter(self.word_list)



Iterator Protocol: Virtual & Subclasshook
------------------------------------------

...