==================
Exception Handling
==================


As stated in the core principles (see :doc:`core-principles`), it is fine to
use exceptions *within* a component. However, it is *not fine* to throw
exceptions beyond component boundaries. Here are a few details on why and how to
use exceptions properly.


Events
======

**Do not enclose event triggers in a try/catch block.** It simply won't work.
Well, it may work in some cases, but there is no guarantee. That's because
the receiver of the event might run asynchronously in a different
thread, in a different process, or even on a different machine --- you cannot
know.

If the receiver throws an exception, there is no reliable way to communicate
this fact to you in the form of exception handling. At the time the exception
is thrown, you may already have left the try/catch block because of
asynchronicity.

So, if you use try/catch with events and it works, please realize that this is
only a coincidence and it is only a matter of time until your design fails. If
you must communicate error information, either use exception events or think about
whether what you are doing really requires an event. In some cases, a request
might be a better choice.

Of course, this entails that **you should never throw exceptions beyond your
component boundary when handling an event**: there is no guarantee that the
exception can be delivered to the sender of the event.


Requests
========

The ``submit`` method of the ``Request`` class returns a ``PortsFuture`` whose
various getters may throw an ``ExecutionException``. The ``call`` method of
``Request`` directly returns a response, but still may throw this exception, too.

An ``ExecutionException`` indicates
that the receiver of the request terminated with an exception and therefore
did not provide a response. Catching this exception works regardless of whether
the receiver handles the request asynchronously or synchronously.

In case of ``submit``, typical code looks like this:

.. code-block:: java

  PortsFuture<Response> future = request.submit(...);
  
  ...
  
  try {
      Response reponse = future.get();
      ...
  } catch (ExecutionException e) {
      ...
  }

And in case of ``call``:
  
.. code-block:: java

  try {
      Response response = request.call(...);
  } catch (ExecutionException e) {
      ...
  }

It is important to understand that **this mechanism is not meant for error
handling** but for being notified that waiting for a response makes no sense
anymore because the receiver has crashed. Of course, this should not happen in
practice; if it does, this means that the receiver lacks proper error handling
which should be implemented.

The ``ExecutionException`` is a ``RuntimeException`` which you don't need to catch.
Oftentimes, there really is no good recovery strategy because you just don't
know what's going on. Handling this exception is of course necessary in cases where
you would do extra damage if you didn't, but in general, you will have a problem if
it occurs, even if you handle it. In any case, error handling of the receiver
should be improved because that's the only real way to mitigate this problem.

If you need to communicate error or status information, use a request
that returns an ``Either`` and follow the core principles (see :doc:`core-principles`) which
mandate that each component is responsible for handling its own errors.


Exception Streamlining
======================


