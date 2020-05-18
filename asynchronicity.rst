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
That's because Ports does not want the developer having to care about asynchronicity
while writing business code. Instead, Ports introduces the more abstract notion of
**synchronzation domains**.

A synchronization domain is an imaginary collection of components that share a common
**dispatch policy** and **synchronization policy**. An application can define
arbitrarily many synchronization domains, but each component can only belong to exactly
one synchronization domain.


Dispatch Policies
-----------------

There are three *dispatch policies*:

#. ``DispatchPolicy.SYNCHRONOUS`` specifies that messages are dispatched synchronously within
   the original threads of their respective senders.
#. ``DispatchPolicy.ASYNCHRONOUS`` specifies that messages are dispatched asynchronously within
   a single separate thread.
#. ``DispatchPolicy.PARALLEL`` specifies that messages are dispatched in parallel within an
   indeterminate number of separate threads. The number of threads depends on the number of logical
   cores available to the virtual machine.

.. WARNING::
   When you choose a ``SYNCHRONOUS`` or ``ASYNCHRONOUS`` dispatch policy, this can still result
   in a higher level of asynchronicity than you might expect. For example, this happens if you
   are using a servlet container like Tomcat
   that automatically creates a number of threads for handling HTTP requests. You can solve this
   problem by forwarding incoming HTTP requests to an IN port of a Ports component as soon as
   possible. This transfers asynchronicity control to Ports.


Synchronization Policies
------------------------

There are three *synchronization poicies*:


