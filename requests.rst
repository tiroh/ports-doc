=========================
How to Write Request Code
=========================

Ports provides four options for sending requests: calls, submits, submit chains,
and forks.

In the following, let's assume that you want to make three requests in a row,
namely ``FirstRequest``, ``SecondRequest``, and ``ThirdRequest``.


Option 1: The Call
==================

The ``call`` method of the ``Request`` class enables you to send a single request
and retrieve the result immediately:

.. code-block:: java

  FirstResponse firstResponse = firstRequest.call(new FirstRequest());

Simple enough, and indeed, the ``call`` method is well-suited to the situation where
you want to send just a single request.

However, in principle, the receiver could fail with an exception which is
relayed to you in the form of an ``org.timux.ports.ExecutionException``, so you
have to use try/catch:

.. code-block:: java

  try {
      FirstResponse firstResponse = firstRequest.call(new FirstRequest());
  } catch (ExecutionException e) {
      ...
  }
  
Now, we want to send the other two requests. It could look like this:

.. code-block:: java

  try {
      FirstResponse firstResponse = firstRequest.call(new FirstRequest());
      ...
      SecondResponse secondResponse = secondResponse.call(new SecondResponse());
      ...
      ThirdResponse thirdResponse = thirdResponse.call(new ThirdResponse));
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
  
  SecondResponse secondResponse;
  
  try {
      ... // use firstResponse here
      secondResponse = secondResponse.call(new SecondResponse());
  } catch (ExecutionException e) {
      ...
  }
  
  ThirdResponse thirdResponse;
  
  try {
      ... // use thirdResponse here
      thirdResponse = thirdResponse.call(new ThirdResponse));
  } catch (ExecutionException e) {
      ...
  }

This example shows that using the ``call`` method can lead to lengthy and cumbersome
code when you have to send multiple requests.


Option 2: The Single Submit
===========================


Option 3: The Submit Chain
==========================

Oftentimes, you want to make multiple requests one after another. Typically, this
involves checking for errors along the way, which can make the code rather lengthy
and cumbersome. Using ``submit`` and the ``Either`` type, you can make this a bit easier:

.. code-block:: java

  firstRequest.submit(new FirstRequest())
           .andThenR(firstResponse -> secondRequest.submit(new SecondRequest())
           .andThenR(secondResponse -> thirdRequest.submit(new ThirdRequest())
           .finallyDo(thirdResponse -> { System.out.println("handle the response"); })
           .orElse(throwable -> { System.out.println("let's handle the problem"); });

The ``andThenR`` method will execute the provided function if no error has occurred
in the preceding requests. Otherwise, it will do nothing and just pass control over
to the next method in the chain.

The ``orElse`` method does the opposite of ``andThenR``: it executes the provided
function if an error has occurred, otherwise it does nothing.

The ``andThenR`` method is named this way because it is a special "request version"
of ``andThen`` (technically, it is just syntactic sugar). It means "and then request".


Option 4: The Fork
==================

Bla.
