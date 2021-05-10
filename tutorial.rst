===================
Ports Tutorial
===================


Step 1: Maven Configuration
===============================================

If your project uses the Vaadin/Spring tandem, include the following Maven
dependency in your ``pom.xml``:

.. code-block:: xml

  <dependency>
      <groupId>org.timux.ports</groupId>
      <artifactId>ports-vaadinspring</artifactId>
      <version>${ports.version}</version>
  </dependency>

Please don't forget to define ``${ports.version}``.

If you do *not* use Vaadin/Spring, you can use the ``core`` module of Ports.
However, this can be a bit tedious, because in its current development stage
it does not provide an IOC container (like Spring does). This means that you
have to instantiate and wire your components manually. Anyway, it works, and
you can import it the following way:

.. code-block:: xml

  <dependency>
      <groupId>org.timux.ports</groupId>
      <artifactId>ports-core</artifactId>
      <version>${ports.version}</version>
  </dependency>



Step 2: Understand the Four Message Types
=========================================

The "Ports way" of seeing the world is as a collection of **independent
agents** communicating via **messages**. For Ports, there is no such thing as a
"method call" or a "dependency". There are only *messages* and the *types* those
messages are referring to. Messages are always of one of the following kinds:

* **Events** are messages with fire-and-forget semantics. They tell the world that
  "something" has happened. This "something" must be a notion that originates from the
  concern of the sender, *not* the receivers. The receivers may react in whatever
  way they want, the sender must not make any assumptions about that. The
  sender-receiver relation is 1:*n*.
* **Requests** are messages with request-response semantics. The receiver should
  ideally react in a functional way, that is, there should be no significant side effects.
  (For example, making a log entry is a side effect, but in this context we do not
  consider it a significant side effect.) When sending a request, the sender *must not*
  make any assumptions about the system state because requests are about *data*,
  not *state*. The sender-receiver relation is 1:1.
* **Commands** are messages that instruct a receiver to modify the system state.
  They are the imperative dual to requests.
  For example, instructing a backend service to save or delete a certain item
  to/from the database is a command, not a request. Like requests, commands have
  request-response semantics because they must return a status report.
  The sender-receiver relation is 1:1.
* **Exceptions** are messages with fire-and-forget semantics that tell the world
  that the system has entered an *undefined state*. This sounds scary but if basic
  rules are followed (see :doc:`core-principles` and :doc:`best-practices`),
  oftentimes it can be avoided that the consequences
  manifest themselves in a crash or in a corrupt database. The sender-receiver
  relation is 1:*n*.


Step 3: Learn about the Core Principles and Best Practices
==========================================================

If you just want to set everything up, you can skip this step for now.

However, if you are using Ports in a project, you need to get familiar with
the ideas and concepts that form the foundation of the framework. Otherwise, you
probably won't be able to make
good use of it. So, it is a good idea if you make some time for having a
thorough look at these pages:

* :doc:`core-principles`
* :doc:`best-practices`


Step 4: Setup Your Application
==============================

If you are using Ports in a Vaadin/Spring application, you have to apply the
``@EnablePorts`` annotation to your Spring application class.

If you are using plain Ports (only the ``core`` module), there is no setup
required.


Step 5: Create Message Types
============================

Your application will need message types, so let us create some of them.

This is how you create events and simple requests:

.. code-block:: java

  public class SomethingHappenedEvent {
  }

.. code-block:: java

  public class MyFailureException {   // could also extend RuntimeException, if you want to
  }

.. code-block:: java

  @Response(Integer.class)            // declares what type the response will have
  public class DefaultNumberRequest {
  }

A request or command can also have two or three response types. Only one of them
will be returned. This works using the ``Either`` and ``Either3`` types (see
:doc:`requests`, :doc:`exception-handling`, and :doc:`types` for details).
The main use case is for indicating a success or failure, like this:

.. code-block:: java

  @Response(FindDataResponse.class)   // these two types must be returned
  @Response(Failure.class)            // using an Either
  public class FindDataRequest {
  }

.. code-block:: java

  @Response(Success.class)
  @Response(Failure.class)
  public class SaveDataCommand {
  }

The event types and request types must have the indicated suffixes ("Event",
"Exception", "Request", "Command"). The framework enforces this naming convention.

You can also use multiple return types in scenarios which are not of the success/failure
type. For example, it might happen that a database request generates one of several
possible data transfer objects, depending on the state of the database, or that
nothing is returned (if nothing is found in the database). So, in general, you
can write something like this:

.. code-block:: java

  @Response(YellowDataResponse.class)      // these three types must be returned
  @Response(GreenDataResponse.class)       // using an Either3
  @Response(Nothing.class)
  public class FindColoredDataOrNothingRequest {
  }

A request whose response comprises multiple types must return an ``Either`` or
an ``Either3`` (see next section).

In order to improve performance, requests can be marked as `pure`. For more information,
please see :doc:`pure-requests`.


Step 6: Send Messages via OUT Ports
===================================

There are two kinds of OUT ports: **event ports** and **request ports**. Event ports are used
for both event and exception messages, and request ports are used for both request
and command messages.

In a component, you declare them as members of the component class marked with the
``@Out`` annotation. Observe that if you have a request or command that returns
multiple types, you have to use the ``Either`` or ``Either3`` types in the declarations
of your OUT ports.


.. code-block:: java

  @Out
  private Event<SomethingHappenedEvent> somethingHappenedEvent;
  
  @Out
  private Event<MyFailureException> myFailureExceptionEvent;
 
  @Out
  private Request<defaultNumberRequest, Integer> defaultNumberRequest;
  
  @Out
  private Request<FindDataRequest, Either<FindDataResponse, Failure>> findDataRequest;
  
  @Out
  private Request<SaveDataCommand, Either<Success, Failure>> saveDataCommand;
  

The member names must follow the naming convention shown. This is enforced by the framework.

You can send events and exceptions using the ``trigger`` method. For requests and commands,
you can use the methods ``call`` (returns the response directly), ``callE`` (returns
the response packaged into an ``Either<T, Failure>``), ``callF`` (returns a
``PortsFuture``), or ``fork`` (sends multiple requests at once and returns a ``Fork``).
It works like this (for details, please see :doc:`requests`):

.. code-block:: java

  somethingHappenedEvent.trigger(new SomethingHappenedEvent());
  myFailureExceptionEvent.trigger(new MyFailureException());
  
  int defaultNumber = defaultNumberRequest.call(new DefaultNumberRequest());
  Either<FindDataResponse, Failure> response = findDataRequest.call(new FindDataRequest());
  Either<Success, Failure> status = saveDataCommand.call(new SaveDataCommand());
  
  PortsFuture<FindDataResponse> future = findDataRequest.callF(new FindDataRequest());
  
  Fork<FindDataResponse> fork = findDataRequest.fork(10, k -> new FindDataRequest();

By default, Ports dispatches all messages synchronously, i.e. there is no
concurrency involved. In order to determine whether message have to be dispatched
synchronously or asynchronously and how synchronization is to be handled, Ports
uses a concept called `synchronization domains`. For more information, please
have a look at :doc:`asynchronicity`.



Step 7: Receive Messages via IN Ports
=====================================

All four message types (events, exceptions, requests, and commands) are
usually received by methods marked as IN ports using the ``@In`` annotation.

The signatures of the IN port methods must match the signatures of the OUT port
members. It looks like this:

.. code-block:: java

  @In
  private void onSomethingHappened(SomethingHappenedEvent event) {
      ...
  }
  
  @In
  private void onMyFailureException(MyFailureException exception) {
      ...
  }
  
  @In
  private Integer onDefaultNumberRequest(DefaultNumberRequest request) {
      return 37;
  }
  
  @In
  private Either<FindDataResponse, Failure> onFindDataRequest(FindDataRequest request) {
     ...
     if (isErrorCondition) {
          return Either.failure(...);
      } else {
          return Either.a(new FindDataResponse(...));
      }
  }
  
  @In
  private Either<Success, Failure> onSaveDataCommand(SaveDataCommand command) {
     ...
     if (isErrorCondition) {
          return Either.failure(...)
      } else {
          return Either.success();
      }
  }
  
There is also the option to receive events and exceptions in IN ports with stack
semantics or queue semantics. It looks like this:

.. code-block:: java

  @In
  private QueuePort<SomethingHappenedEvent> somethingHappenedEventQueue;
  
  @In
  private StackPort<MyFailureException> myFailureExceptionStack;

Using the ``peek``, ``poll``, and ``pop`` methods of ``QueuePort`` and ``StackPort``
(respectively), you can access the received messages. This feature is designed
for so-called *active components* that are not waiting for messages but running
continuously. Under normal circumstances, it is not required, and it is *not recommended*
for regular components.



Step 8: Connect Your Components
===============================


If you are using the Vaadin/Spring tandem, you don't have to connect your
components yourself. Your components are Spring components (marked with either
``@SpringComponent`` or ``@Service``) that are instantiated and managed by the
Spring IOC container. The Ports Framework hooks into this process and connects
all components automatically.

However, if you use plain Ports (the ``core`` module), for example in testing,
you need to create the connections yourself. This is not difficult:

Let's assume you have three components, ``A``, ``B``, and ``C``. You instantiate and
connect them the following way:

.. code-block:: java

  A a = new A();
  B b = new B();
  C c = new C();
  
  Ports.connect(a).and(b);
  Ports.connect(a).and(c);
  Ports.connect(b).and(c);

Yes, this would be tedious if you had many components to connect. In future versions of
Ports, this will become simpler. However, for the time being, that's how it works.
Actually, you *should not* have many components to connect here. If you want to
do integration tests, it is best to use Ports's protocols in order to mock
unconnected OUT ports (see :doc:`protocols`).

The ``and`` method supports ``PortsOptions``. Have a look into the Javadoc
for ``PortsOptions`` for more information.


Step 9: Get Familiar With the Vaadin/Spring Tandem
==================================================

If your application uses the Vaadin/Spring tandem, please read :doc:`vaadin-spring`
for some additional information and best practices.
