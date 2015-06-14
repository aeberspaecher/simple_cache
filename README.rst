simple_cache
============

A caching simple decorator for class member functions in Python.

Why?
----

Sometimes, class member functions may be expensive to call. It may happen that
e.g. during interactive numerical work, the same function is called under
identical conditions more than only once. In that case, it is desirable that
the function's result comes from a cache instead from an expensive
recomputation. However, the cache needs to be aware of the object's state.
That's what ``simple_cache`` is written for.

This work is heavily inspried by Steven Fernandez' `SuPyCache
<https://github.com/lonetwin/supycache>`_, but is tailored to class member
functions as it can deal with ``self``.

Usage
-----

Simply create a string that evaluates to a unique key using format(). To key
needs to describe the relevant object state variables and the relevant function
arguments. E.g. for ``Class`` with state variable ``Class.a`` and the member
function ``f(x)``, the key might be ``{self.a}_{x}`` or a more descriptive
``self.a={self.a}_x={x}``. The keys will however not be exposed to the user, so
simple formats that allow for a unique mapping of all state variables and
arguments to a key are as good more verbose formats.

To get caching capabilities, simply wrap your expensive member function as
follows:

.. code:: python

    obj_state = "{self.state_var1}_{self.state_var2}"  # describes object state

    class TestObject(object):
        def __init__(self, var1, var2):
            self.state_var1, self.state_var2 = var1, var2

        @simple_cache.cache(key_template=obj_state + "_{func_arg})
        def expensive_computation(func_arg):
            ...


How does it work?
-----------------

The basic idea is the same as in `SuPyCache
<https://github.com/lonetwin/supycache>`_: on runtime, the decorator calls
``.format(**kwargs)`` on the key and thus allows to evaluate both the class'
state and the function's argument on runtime. All positional ``args`` are
converted to keyword arguments before that. The key produced in that way is
then used with the cache to retrieve cached results; if the key is not
presented, the function is called and the result will be stored for later use.

What about caches?
------------------

Again, similar to `SuPyCache <https://github.com/lonetwin/supycache>`_,
anything that has ``get()``, ``set()`` and ``clear()`` methods can be a cache.
As a default, ``simple_cache`` comes with ``FiniteCache(num_items)``, which
implements a dictionary-like object that holds at most ``num_items`` items.
Note that caching objects are local to the decorated functions.

To use larger ``FiniteCache``, decorate e.g. with:

.. code:: python

    @simple_cache.cache(key_template=...,  get_cacher=simple_cache.FiniteCache(10))

License and copyright
---------------------

Copyright 2015 Alexander Eberspächer

BSD license, see LICENSE file
