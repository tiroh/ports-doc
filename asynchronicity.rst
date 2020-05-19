==============================
Asynchronicity and Parallelism
==============================

Starting with version 0.5.0, Ports supports asynchronicity and
parallelism. In order to make asynchronous programming as simple and transparent
as possible, Ports abstracts from the rather technical notion of multithreading and
introduces its own model of asynchronicity. In many cases, this model enables you to
benefit from asynchronous and/or parallel execution without having to worry about the
intricate complexities of synchronization and deadlock prevention.



Synchronization Domains
=======================

An important difference between Ports and other frameworks is that Ports
does not recognize the concept of a "synchronous" or an "asynchronous" message.
That's because separation of concerns is an important cornerstone of Ports's
philosophy, and a developer having to care about asynchronicity while writing
business code is effectively mixing concerns. So, Ports introduces the more
abstract notion of **synchronzation domains** that allows transparent separation
of asynchronicity from business logic.

A synchronization domain is an imaginary collection of components that share a common
**dispatch policy** and a common **synchronization policy**. An application can define
arbitrarily many synchronization domains, but each component can only belong to exactly
one synchronization domain.


Dispatch Policies
-----------------

There are three *dispatch policies* that define how a domain dispatches messages:

#. ``DispatchPolicy.SYNCHRONOUS`` specifies that messages are dispatched synchronously within
   the original threads of their respective senders.
#. ``DispatchPolicy.ASYNCHRONOUS`` specifies that messages are dispatched asynchronously within
   a single separate thread.
#. ``DispatchPolicy.PARALLEL`` specifies that messages are dispatched in parallel within an
   indeterminate number of separate threads. The number of threads depends on the number of
   logical cores available to the virtual machine.

.. NOTE::
   **Do not parallelize tiny tasks.**
   
   Dispatching messages in parallel necessarily causes some performance overhead.
   If the dispatched tasks are only very small, the aggregated overhead might be
   larger than the speedup.

.. WARNING::
   **A domain must only use parallel dispatching if all of its IN ports are order-agnostic**.
   
   Be aware that the ``PARALLEL`` dispatch policy implies
   out-of-order dispatching of messages. The scheduler of your operating system is
   responsible for managing threads, and it may decide to let them run in arbitrary order.
   (This is *not* mere theory but an everyday reality. For example, the schedulers of
   Linux or Windows *really* do this in practice.)
   
   In general, if you do two ``call``-s to a parallel domain, then from
   the viewpoint of the
   sender their responses appear in order, but from the viewpoint of the receiving domain
   their responses might be reversed. This does not matter if your messages are side-effect
   free, but if not (for example, when they modify the database), the resulting state of
   your system might be undefined or at least "surprisingly different than expected".

.. WARNING::
   **Third-party frameworks relying on thread-local data may not work properly if you choose
   asynchronous or parallel dispatching.**
   
   Some frameworks (like Spring or Vaadin) bind session-related data to specific threads.
   That means, if your application uses asynchronous or parallel domains, that
   session-related data is not available in components running within those domains and
   any calls referring to that data must fail.
   
   To solve this problem, you should make session-related data explicit and pass it
   from one component to another. No component should ever assume that it is
   running within a specific context, because that assumption represents a tight
   architectural coupling that impairs maintainability, distributability, and
   parallelizability.


Synchronization Policies
------------------------

There are three *synchronization policies* that define how a domain handles concurrent
accesses:

#. ``SyncPolicy.NONE`` specifies that messages shall be processed without any synchronization.
   Should there be different threads accessing the domain, Ports will not take any measures of
   synchronizing them.
#. ``SyncPolicy.COMPONENT`` specifies that message processing is subject to mutual exclusion
   w.r.t. to individual components.
#. ``SyncPolicy.DOMAIN`` specifies that message processing is subject to mutual exclusion
   w.r.t. to the complete synchronization domain.

.. WARNING::
   **Third-party frameworks might take a hand in thread management and
   invalidate the synchronization invariants of Ports.**

   If you are using third-party frameworks, this can result in a higher level of
   asynchronicity than you might expect if those frameworks take a hand in thread management.
   
   For example, when using a servlet container like Tomcat, it automatically creates a
   number of threads for handling HTTP requests which makes all of your code inherently
   asynchronous and prone to race conditions. You can solve this
   problem by forwarding incoming HTTP requests to an IN port of a Ports component as soon as
   possible. This transfers asynchronicity control to Ports.


The Default Domain
------------------

By default, each component is assigned to the **default domain** which employs
``DispatchPolicy.SYNCHRONOUS`` dispatching and ``SyncPolicy.COMPONENT`` synchronization.
This ensures that Ports will cooperate out-of-the-box with any third-party framework
messing around with threads.


Making Components Asynchronous
==============================

Let's say you have identified three components ``com.example.mydomain.A``,
``com.example.mydomain.B``, and ``com.example.mydomain.C``
that you want to run asynchronously. For this purpose, you already have moved them
into a common package ``com.example.mydomain``.

First, you have to create a synchronization domain for them. Let us create a domain that
dispatches asynchronously and that synchronizes on component level:

.. code-block:: java

  Domain myDomain = Ports.domain(
      "my-domain",                   // the name of the new domain (mainly for debugging)
      DispatchPolicy.ASYNCHRONOUS,
      SyncPolicy.COMPONENT);

Then, you add the components to the newly created domain. You can define domain membership
in the following three ways:

#. you specify **packages** whose classes shall be members of the domain (incl. subpackages),
#. you specify **classes** whose instances shall be members of the domain,
#. you specify **instances** that shall be members of the domain.

Of course, specifying a package is the most convenient option, so let us try that one:

.. code-block:: java

  myDomain.addPackages("com.example.mydomain");   // not very robust!

But what happens if the package is renamed? Another, safer method could be:

.. code-block:: java

  myDomain.addPackages(A.class.getPackage().getName());   // still not perfect!

Of course, if the components are moved during a (careless) refactoring, the domain
wouldn't work anymore as expected. So, you could create a special, empty class
in the ``com.example.mydomain`` package that is never moved and that is only there
so that you can safely refer to the ``com.example.mydomain`` package, even if its name
should change in the future:

.. code-block:: java

  myDomain.addPackages(MyDomainPackage.class.getPackage().getName());

Alternatively, you could specify the classes individually, which is a very safe approach,
albeit not very flexible:

.. code-block:: java

  myDomain.addClasses(A.class, B.class, C.class);   // pretty robust!

That's it. From now on, all messages sent to those three components will be subject to
the policies of domain ``my-domain``. Of course, keep in mind that if you used the
``addPackages`` method and package
``com.example.mydomain`` contained more components than just ``A``, ``B``, and ``C``,
those additional components would also be members of domain ``my-domain``.

.. NOTE::
   You have to specify how your team manages domains: do you want to work
   with packages or with individual classes? If a component is moved or a new component
   is added to a package, do you want it to leave or enter a domain automatically or
   should this be an explicit decision?

.. NOTE::
   Normally, you want to configure your domains as early as possible during your
   application's startup phase. You may configure them at a later point, but keep
   in mind that the policies of the default domain will be used for all components
   as long as you haven't configured any other domains.


Events
======

The ``Event`` class only provides one method for sending messages: the ``trigger``
method. Depending on the receiver's synchronization domain, this method could
return immediately, even before the event has been processed, or it could block
until the event has been processed. The sender cannot know and must not assume
anything. Remember that events are messages with fire-and-forget semantics.

In particular, this means that **you must not enclose an event trigger in a try/catch
block**. See :doc:`exception-handling` for more details on this topic.


Requests, Futures, and Forks
============================

The ``Request`` class provides three methods for sending messages (in addition to
futher methods that are convenience variants of those three):

#. ``call``,
#. ``submit``,
#. ``fork``.

The difference between ``call`` and ``submit`` is that ``call`` waits for a response
(if necessary) and returns it directly, while ``submit`` does not wait but returns a
``PortsFuture`` instead. ``PortsFuture`` implements Java's ``Future`` interface and
provides facilities to handle failures without exceptions (by using the union types
``Either`` and ``Either3``).
The ``call``  method can be regarded as syntactic sugar for a ``submit`` followed
by an immediate ``get`` on the returned ``PortsFuture``.

The ``fork`` method issues multiple ``submit``-s at once. It does not wait for a
response, but returns a ``Fork`` instance, which also implements Java's ``Future`` interface.
The ``fork`` method provides support for the well-known Fork-Join pattern of
parallel computing.

It is important to understand that **the sender of a message has no control whatsoever
about whether the message will be handled synchronously, asynchronously, or in parallel**.
It is completely up to the receiver's synchronization domain to decide how messages are
handled.
So, for example, if you use the ``fork`` method, it may be that all your forked
requests are handled synchronously if the receiver's domain does not allow asynchronous
or parallel dispatch.

.. NOTE::
   The ``fork`` method provides maximum performance improvement only when the receiver's
   synchronization domain supports ``PARALLEL`` dispatch with synchronization level ``NONE``.


Deadlocks
=========

In a distributed, concurrent system, deadlocks are an ubiquitous hazard. They happen
when a component **A** waits for the response of another component **B** (possibly via
a chain of intermediate messages) that in turn waits for a response of component **A**.
In this situation, **A** cannot deliver a response to **B** because **A** is locked and
hence cannot accept **B**'s request; **A**'s
lock will only be released when it receives a response from **B**, which will never
happen because **B** is waiting for a response from **A**. Therefore, the system enters an
everlasting wait state --- a *deadlock*. Let's call this kind of circular
relationship between components a **critical loop**.

Critical loops happen very quickly in practice, so you should take them into account when
designing synchronization domains.

Of course, avoiding critical loops can be difficult, and it may happen that you do
your best but still have a critical loop
in your system that is not discovered during testing. That's why Ports has the ability
to detect and resolve deadlocks at runtime by temporarily lifting the synchronization
invariant for exactly that thread that is causing the deadlock. This is safe
in the sense that there is exactly one possible order of execution.

However, you should not rely solely on Ports's deadlock resolution. The reason
is that Ports can only guarantee that your system does not enter an eternal wait state,
but it cannot guarantee that your system's state remains correct.
Lifting the synchronization invariant even for a single request means that there
is a potential source of undefined state should your code strongly rely on upholding
the synchronization invariant at all times.

So, you should always try to avoid critical loops in your system, even if Ports can
detect and resolve deadlocks automatically. Also, there is a performance penalty associated
with resolving deadlocks; you won't notice it if you only have a few deadlocks per
second, but if you have, say, 1000 deadlocks per second, there could be some
noticeable performance degradation. 
