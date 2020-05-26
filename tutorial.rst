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

Your application will need message types, so let us create four of them:

.. code-block:: java

  public class SomethingHappenedEvent {
  }

.. code-block:: java

  @Response(FindDataResponse.class)   // declares what type the response will have
  public class FindDataRequest {
  }

.. code-block:: java

  @Response(SaveDataStatus.class)
  public class SaveDataCommand {
  }

.. code-block:: java

  public class MyFailureException {   // could also extend RuntimeException, if you want to
  }

The event types and request types must have the indicated suffixes ("Event",
"Exception", "Request", "Command"). The framework enforces this naming convention.

A request or command can also have two response types. Only one of them will be returned.
The main use case is for indicating a success or failure, like this:

.. code-block:: java

  @SuccessResponse(FindDataResponse.class)
  @FailureResponse(ErrorCode.class)
  public class FindDataRequest {
  }

.. code-block:: java

  @SuccessResponse(SaveDataResponse.class)
  @FailureResponse(ErrorCode.class)
  public class SaveDataCommand {
  }

Sometimes, you might want to return one of two types without being in the success/failure
scenario. In this case, you can use two ``@Response`` annotations:

.. code-block:: java

  @Response(YellowDataResponse.class)
  @Response(GreenDataResponse.class)
  public class FindColoredDataRequest {
  }


Step 6: Send Messages via OUT Ports
===================================

There are two kinds of OUT ports: **event ports** and **request ports**. Event ports are used
for both event and exception messages, and request ports are used for both request
and command messages.

In a component, you declare them as members of the component class marked with the
``@Out`` annotation. It looks like this:


.. code-block:: java

  @Out
  private Event<SomethingHappenedEvent> somethingHappenedEvent;
 
  @Out
  private Request<FindDataRequest, FindDataResponse> findDataRequest;
  
  @Out
  private Request<SaveDataCommand, SaveDataStatus> saveDateCommand;

  @Out
  private Event<MyFailureException> myFailureExceptionEvent;

If you have a request or command port that returns a union type (see above),
you declare the OUT ports like this:

.. code-block:: java

  @Out
  private Request<FindDataRequest, Either<FindDataResponse, ErrorCode>> findDataRequest;
  
  @Out
  private Request<SaveDataCommand, Either<SaveDataResponse, ErrorCode>> saveDataCommand;

The member names must follow the naming convention shown.
This is enforced by the framework.

You can send events and exceptions using the ``trigger`` method. For requests and commands,
you can use either the ``call``, ``submit``, or ``fork`` methods:

.. code-block:: java

  somethingHappenedEvent.trigger(new SomethingHappenedEvent());
  myFailureExceptionEvent.trigger(new MyFailureException());
  
  FindDataResponse response = findDataRequest.call(new FindDataRequest());
  SaveDataStatus status = saveDataCommand.call(new SaveDataCommand());
  
  PortsFuture<FindDataResponse> future = findDataRequest.submit(new FindDataRequest());
  
  Fork<FindDataResponse> fork = findDataRequest.fork(10, k -> new FindDataRequest();

Up to Ports 0.4.1, all messages are handled synchronously. Starting with
Ports 0.5.0, so-called **synchronization domains** are used to determine how
messages are handled. For more information on asynchronicity and parallelism, please
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
  private FindDataResponse onFindDataRequest(FindDataRequest request) {
     ...
     return new FindDataResponse(...);
  }
  
  @In
  private SaveDataStatus onSaveDataCommand(SaveDataCommand command) {
     ...
     return new SaveDataStatus(...);
  }
  
  @In
  private void onMyFailureException(MyFailureException exception) {
     ...
  }

If you have a request or command port that returns a union type (see above),
you declare the IN ports like this:

.. code-block:: java

  @In
  private Either<FindDataResponse, ErrorCode> onFindDataRequest(FindDataRequest request) {
      ...
      if (isErrorCondition) {
          return Either.b(new ErrorCode(...));
      } else {
          return Either.a(new FindDataResponse(...));
      }
  }
  
  @In
  private Either<SaveDataResponse, ErrorCode> onSaveDataCommand(SaveDataCommand command) {
      ...
      if (isErrorCondition) {
          return Either.b(new ErrorCode(...));
      } else {
          return Either.a(new SaveDataReponse(...));
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
