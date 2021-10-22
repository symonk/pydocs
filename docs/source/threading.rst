Concurrency: Threading
=========================

Threading: Introduction
------------------------

Before we start a deep dive into python threading, a core concept to understand is that python
in the context of (CPython) has a **Global Interpreter Lock** (GIL).  Certain performance centric
libraries are able to overcome this limitation but it often involves moving into `C` extensions
etc.  When wondering if threading is the right answer, consider:

    * Moving to `Processing` if you find yourself CPU bound and wish to benefit from multiple cores.
    * Use threading if you find yourself IO blocked and want to open up some simultaneous execution of those tasks.

