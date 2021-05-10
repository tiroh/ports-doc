=============================
Four Ways Of Sending Requests
=============================

Ports provides four options for sending requests:

* immediate calls,
* chained calls,
* future calls,
* forked calls.

In the following, we demonstrate how each option works. Let's assume
you want to make three requests in a row,
namely ``FirstRequest``, ``SecondRequest``, and ``ThirdRequest``, and that
each of them depends on the result of the respective request before it.


Immediate Calls
===============

The ``call`` method of the ``Request`` class enables you to send a single request
and retrieve the result immediately:

.. code-block:: java

  FirstResponse firstResponse = firstRequest.call(new FirstRequest());

Simple enough, and indeed, the ``call`` method is well-suited to the situation where
you want to send just a single request.

However, in principle, the receiver could fail with an exception. This exception
would be relayed to you in the form of an ``org.timux.ports.ExecutionException``. This is because
the response type of the request is just ``FirstResponse`` instead of
``Either<FirstResponse, Failure>`` in which case any potential exception could
be streamlined into the ``Failure`` branch of the ``Either`` (see
`Exception Streamlining`_ for more details on this topic).

So, because you only have a ``FirstResponse`` instead of an ``Either<FirstResponse, Failure>``,
you could use the traditional try/catch block:

.. code-block:: java

  try {
      FirstResponse firstResponse = firstRequest.call(new FirstRequest());
  } catch (ExecutionException e) {
      ...
  }

However, this kind of code is cumbersome to write and maintain. Luckily, there is
a way to generate an ``Either<FirstReponse, Failure>`` using the ``callE``
method of the ``Request`` class (``callE`` means "call either"):

.. code-block:: java

  Either<FirstResponse, Failure> either = firstRequest.callE(new FirstRequest());

  either.on(
      firstResponse -> { /* handle the response */ },
      failure -> { /* handle the failure */ }
  );

The ``Failure`` branch of the ``Either`` catches the exception for you and streamlines
it so that you can process it just like any other return value.
  
Now, we want to send two more requests, ``SecondRequest`` and ``ThirdRequest``,
which depend on the response of the respective previous request.
It could look like this:

.. code-block:: java

  try {
      FirstResponse firstResponse = firstRequest.call(new FirstRequest());
      ...
      SecondResponse secondResponse = secondResponse.call(new SecondRequest());
      ...
      ThirdResponse thirdResponse = thirdResponse.call(new ThirdRequest());
  } catch (ExecutionException e) {
      ...
  }

But what if we had to react differently to individual failures of the requests? We
would need code like this:

.. code-block:: java

  FirstResponse firstResponse;

  try {
      firstResponse = firstRequest.call(new FirstRequest());
  } catch (ExecutionException e) {
      ...
  }

  if (firstResponse != null) {
      SecondResponse secondResponse;
  
      try {
          ... // use firstResponse here
          secondResponse = secondResponse.call(new SecondResponse());
      } catch (ExecutionException e) {
          ...
      }

      if (secondResponse != null) {
          ThirdResponse thirdResponse;
  
          try {
              ... // use thirdResponse here
              thirdResponse = thirdResponse.call(new ThirdResponse));
          } catch (ExecutionException e) {
              ...
          }
      }
  }

This code is pretty difficult to read and maintain. You might think that using
``callE`` wouldn't make it any better because it would result in an abomination like
this:

.. code-block:: java

  Either<FirstResponse, Failure> either1 = firstRequest.callE(new FirstRequest());

  either1.on(
      firstResponse -> {
          Either<SecondResponse, Failure> either2 = secondRequest.callE(new SecondRequest());

          either2.on(
              secondResponse -> {
                  Either<ThirdResponse, Failure> either3 = thirdRequest.callE(new ThirdRequest());
                  
                  either3.on(
                      thirdResponse -> { ... },
                      failure3 -> { ... }
                  );
              },
              failure2 -> { ... }
          );
      },
      failure1 -> { ... }
  );

Don't worry, there is a better way, namely *chained calls*.


Chained Calls
=============

The above task of sending three requests and handling their individual failures
can also be accomplished in the following way:

.. code-block:: java

  firstRequest.callE(new FirstRequest())
          .orElseDoOnce(failure -> { ... })
          .andThen(firstResponse -> secondRequest.callE(new SecondRequest())
          .orElseDoOnce(failure -> { ... })
          .andThen(secondResponse -> thirdRequest.callE(new ThirdRequest())
          .orElseDoOnce(failure -> { ... })
          .andThenDo(thirdResponse -> { ... });

This code style can be more concise than the code above with all the exception handling.

The ``andThen`` method will execute the provided function if no error has occurred
in the preceding requests. Otherwise, it will do nothing and just pass control over
to the next method in the chain.

The ``orElseDoOnce`` method does the opposite of ``andThen``: it executes the provided
function if an failure has occurred, otherwise it does nothing. It will not handle
a failure that has already been handled by an earlier ``orElse``, ``orElseDo``, or
``orElseDoOnce``. (The ``orElse`` or ``orElseDo`` methods, however, *will* handle a
failure that has already been handled.)

Apart from ``andThen`` and ``orElseDoOnce``, the ``Either`` and ``Either3`` types
provide still more functions for handling chains in different kinds of scenarios.
Have a look at their Javadoc, they are simple to use. 


Future Calls
============

In some cases, you might want to improve the runtime of a certain procedure by
executing a time-consuming request asynchronously. For this, you have to place
the request's receiver in an asynchronous or parallel domain
(see :doc:`asynchronicity`) and then use the
``callF`` method of the ``Request`` class (``callF`` means "call future"):

.. code-block:: java

  PortsFuture<FirstResponse> future = firstRequest.callF(new FirstRequest());
  
  ...   // do something else while the request is being executed
  
  FirstResponse response = future.get();   // finally retrieve the result

The ``get`` call will return the result immediately if it is available. If it is
not available, it will block until it is.

Note that a request (or event, for that matter) will only be dispatched
asynchronously or in parallel if its receiver is placed within an asynchronous
or parallel domain. Otherwise, the ``callF`` method will block. (The ``callF``
method will also block if the receiver's dispatch capacity has been reached.)

The ``PortsFuture`` class provides more functions than just ``get``. For example,
the ``getE`` function wraps the result in an ``Either<T, Failure>``. Also, there
are several variants of ``map``, ``andThen``, and ``orElse`` available.



Forked Calls
============

Using the ``fork`` method, you can send multiple requests at once and potentially
have them dispatched asynchronously or in parallel (they must
all be of the same type):

.. code-block:: java

  // send 10 requests:
  Fork<FirstResponse> fork = firstRequest.fork(10, k -> new FirstRequest());
  
  ...   // do something else while the requests are being executed
  
  List<FirstResponse> responses = fork.get();   // finally retrieve the results

The ``get`` method blocks until all results are available and returns them in
a list. There are  other variants of ``get`` available. For example, you can
retrieve a list of ``Either`` instead of a list of results.

Also, the ``Request`` class provides still more ``fork`` variants then the one shown
here.

Note that the requests are only executed asynchronously or in parallel if the
receiver is placed in an asynchronous or parallel domain. See :doc:`asynchronicity`
for details.


Pure Requests
=============

In order to improve performance, requests can be marked as `pure`. Please see
:doc:`pure-requests` for details.


Exception Streamlining
======================

If you have a request that returns an ``Either<T, Failure>``, you wouldn't want
to use the ``callE`` method since it would return an ``Either<Either<T, Failure>, Failure>``.
Most of the time, a type like that is hard to handle and not desirable.
Luckily, in this scenario you can
just use ``call``: it will streamline any exception that should be thrown into the
``Failure`` branch of the ``Either``. This also works with ``Either3<T1, T2, Failure>``.
The only thing to keep in mind is that the ``Failure`` branch must always be the last
type argument.
