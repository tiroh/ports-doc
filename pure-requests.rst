============================
Pure and Quasi-Pure Requests
============================

Ports supports caching of responses in order to improve performance. For this
purpose, we need to introduce the concepts or *pureness* and *quasi-pureness*
which in turn require the concepts of *significant* and *non-significant
side effects*.

.. NOTE::
   This chapter is mainly about performance optimization, so if your performance
   is perfect, it is not essential to read on. However, pureness and quasi-pureness
   are important concepts that you should think about even when performance is not
   your primary concern, so I encourage you to continue.


Significant and Non-Significant Side Effects
============================================

A **significant side effect** (or **essential side effect**)
is a side effect that must happen in order to ensure correctness. Put
differently, a significant side effect is a side effect that is important enough
so that it would be a bug were it omitted. A typical example for this kind of
side effect is a modification of the database.

A **non-significant side effect** is an "optional" side effect.
Put differently, a "non-significant side effect" is a side effect that may be
omitted; neither the system nor the user will care. A typical example for this
kind of side effect is a log entry.


Pureness and Quasi-Pureness
===========================

* A **quasi-pure request** is a request that does not have significant side effects.
* A **pure request** is a quasi-pure request that for a given input always responds
  with the same output.

It follows that commands can neither be pure nor quasi-pure because by definition,
they modify the system state in a significant way.


Exploiting Pureness
===================

Whenever a request is pure, we may cache its response in order to improve performance.

For instance, consider the following request:

.. code-block:: java

  @Response(Integer.class)
  public class DefaultNumberRequest {
  }
  
Let us assume it has the following implementation:

.. code-block:: java

  @In
  private Integer onDefaultNumberRequest(DefaultNumberRequest request) {
    return 37;
  }

In this case, we see that ``DefaultNumberRequest`` is a pure request, since (a)
it always returns the same value (it doesn't have any input) and (b) it doesn't
have any side effects. We can inform Ports about this fact by applying the ``@Pure``
annotation to the request type:

.. code-block:: java

  @Response(Integer.class)
  @Pure
  public class DefaultNumberRequest {
  }

Now, this request's response will be cached. The code of the IN method will only
be executed once. Even in a trivial case like this that is a good thing because
it avoids overhead that is associated with message dispatch, especially when you
have an asynchronous setup (see :doc:`asynchronicity`).


.. NOTE::
   Ports's response caching is meant to be a lightweight optimization that
   accelerates request chains or loops in which certain requests are sent over
   and over again.
   However, it is *not* meant to be a one-size-fits-all caching mechanism for
   all your application's data. Ports's response caches are rather small and
   optimized for minimal overhead. If your application handles large amounts of
   complex data, you still should look for a specialized solution.


Exploiting Quasi-Pureness
=========================

Unfortunately, in a real-world system, most non-trivial requests are not pure,
even when they don't have significant side effects. That is because in general,
they don't respond with the same output given the same input.

For instance, when you retrieve a certain static object from the database using a
request, you might think that this request is pure because (a) it
always returns the same object and (b) it does not have significant
side effects. Well: (b) is true, but (a) is not. When you call
this request multiple times, it might return different data because in the
meantime between calls the object might have been modified. However, in this
scenario, we might notice that the request is quasi-pure since
it doesn't have significant side effects. We can exploit this fact the following
way.

Consider a request ``GetObjectRequest`` which fetches a static object from the
database. Observe how we use the ``@Pure`` annotation in order to declare
messages that invalidate the cache:

.. code-block:: java

   @Response(GetObjectResponse.class)
   @Response(Failure.class)
   @Pure(clearCacheOn = {
     ObjectModifiedEvent.class,
     ObjectDeletedEvent.class)
   }
   public class GetObjectRequest {
   }

You can use any message type (events, exceptions, requests, commands)
for invalidating response caches. Whenever one of those messages is sent,
the respective response caches will be cleared.

.. WARNING::
   It is tempting to declare as many requests as possible pure, but remember that
   your team needs a very thorough and strict policy in order to get the
   cache invalidation right. When in doubt, use the ``@Pure`` annotation only on
   pure requests, but not on quasi-pure requests.


Requests with Parameters
========================

In the previous examples, we only considered requests that have no parameters.
Let us now have a look at a request called ``GetObjectByIdRequest`` that fetches
an object from the database that is identified by an ID.

In this case, we must implement both ``equals`` and ``hashCode`` so that the
cache can determine whether it already possesses the right response or not:

.. code-block:: java

   @Response(GetObjectByIdResponse.class)
   @Response(Failure.class)
   @Pure(clearCacheOn = {
     ObjectModifiedEvent.class,
     ObjectDeletedEvent.class)
   }
   public class GetObjectByIdRequest {
   
     private final long id;
     
     public GetObjectRequest(long id) {
       this.id = id;
     }
     
     public long getId() {
       return id;
     }
     
     @Override
     public boolean equals(Object o) {
       if (this == o) return true;
       if (o == null || getClass() != o.getClass()) return false;
       GetObjectByIdRequest that = (GetObjectByIdRequest) o;
       return id == that.id;
     }

     @Override
     public int hashCode() {
       return Long.hashCode(id);
     }
   }

.. WARNING::
   Implementing ``equals`` and ``hashCode`` manually is extremely error-prone.
   In addition, bugs in this area usually have very deep consequences that can
   lead to very confusing failure modes. Do not
   implement those two methods manually but use your IDE's code generation for
   that purpose.

Whenever you use the ``@Pure`` annotation on a stateful request (i.e., a request
that contains non-static, non-final fields), you must implement ``equals`` and
``hashCode``.

.. TIP::
   You do not need to remember when to implement ``equals`` and ``hashCode``
   because Ports will notify you in case you forget. In general, you can just
   declare a request pure and compile. If you get no error, that means you don't
   need those methods.
