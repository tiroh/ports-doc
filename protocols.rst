=========
Protocols
=========

Protocols enable you to inject **(a)** metaevents
(events that trigger when an event or request is triggered) and
**(b)** messages into the system without involvement of the individual components.

These features are particularly useful for testing.
In principle, you can also use them in production for monitoring, logging,
or debugging.


Protocol Setup
==============

If your application uses Vaadin/Spring, you don't need to do any setup.

However, if you use plain Ports (the ``core`` module), you have to register
the components that are supposed to participate in the protocol execution
like this:

.. code-block:: java

  // At some place, you will instantiate your components:
  SomeComponent someComponent = new SomeComponent();
  SomeOtherComponent someOtherComponent = new SomeOtherComponent();
  
  // Then, you may or may not connect them, depending on the components:
  Ports.connect(someComponent).and(someOtherComponent);
  
  // Finally, you register them:
  Ports.register(someComponent, someOtherComponent);


Protocol Cleanup
================

Sometimes, you may want to remove all declared protocols from Ports's
registry. You can do this the following way:

.. code-block:: java

  Ports.releaseProtocols();

This is a compulsory requirement when using protocols in testing, as explained below.


Injecting Metaevents
====================

Let's assume you have the following two OUT ports:

.. code-block:: java

  @Out
  private Event<SomethingHappenedEvent> somethingHappenedEvent;
 
  @Out
  private Request<FindDataRequest, FindDataResponse> findDataRequest;

Protocols always work with *port signatures*, never with ports themselves. So,
you can react to the following metaevents:

* trigger of ``SomethingHappenedEvent``,
* call of ``FindDataRequest``,
* response of ``FindDataRequest``.

This can be done the following way:

.. code-block:: java

  Ports.protocol()
      .when(SomethingHappenedEvent.class)
          .triggers()
              .do_(event -> logger.log("The event happened: {}", event);
  
  Ports.protocol()
      .when(FindDataRequest.class, FindDataResponse.class)
          .requests()
              .do_(request -> logger.log("The request was called: {}", request);
  
  Ports.protocol()
      .when(FindDataRequest.class, FindDataResponse.class)
          .responds()
              .do_(response -> logger.log("The request responds: {}", response);

You can also chain this up in a single protocol:

.. code-block:: java

  Ports.protocol()
      .when(SomethingHappenedEvent.class)
          .triggers()
              .do_(event -> logger.log("The event happened: {}", event);
      .when(FindDataRequest.class, FindDataResponse.class)
          .requests()
              .do_(request -> logger.log("The request was called: {}", request);
      .when(FindDataRequest.class, FindDataResponse.class)
          .responds()
              .do_(response -> logger.log("The request responds: {}", response);

Of course, creating long chains could impair readability.

You may provide predicates that narrow the scope of your metaevents. Let's assume
you want the metaevents to trigger only when the messages contain a numeric payload
that has a certain minimum value:

.. code-block:: java

  Ports.protocol()
      .when(SomethingHappenedEvent.class)
          .triggers(event -> event.getPayload() > 5)
              .do_(event -> logger.log("The event has a payload > 5: {}", event);
      .when(FindDataRequest.class, FindDataResponse.class)
          .requests(request -> request.getPayload() > 7)
              .do_(request -> logger.log("The request has a payload > 7: {}", request);
      .when(FindDataRequest.class, FindDataResponse.class)
          .responds(response -> response.getPayload() > 9)
              .do_(response -> logger.log("The request sends a response > 9: {}", response);


Injecting Responses
===================

Let's assume we have the same request port as above:

.. code-block:: java

  @Out
  private Request<FindDataRequest, FindDataResponse> findDataRequest;

We can override any IN port for this request by injecting a response using a
protocol like this:

.. code-block:: java

  Ports.protocol()
      .when(FindDataRequest.class, FindDataResponse.class)
          .requests()
              .respond(new FindDataResponse());

You can also access the request object, if you wish:

.. code-block:: java

  Ports.protocol()
      .when(FindDataRequest.class, FindDataResponse.class)
          .requests()
              .respond(request -> new FindDataResponse(request.getSomeData()));


Injecting Messages
==================

As above, let's assume you have the following two OUT ports:

.. code-block:: java

  @Out
  private Event<SomethingHappenedEvent> somethingHappenedEvent;
 
  @Out
  private Request<FindDataRequest, FindDataResponse> findDataRequest;

Using the port signatures, you can inject messages the following way:

.. code-block:: java

  Ports.protocol()
      .with(SomethingHappenedEvent.class)
          .trigger(new SomethingHappenedEvent());
  
  Ports.protocol()
      .with(FindDataRequest.class, FindDataResponse.class)
          .call(new FindDataRequest());

In this case, both ``with`` blocks are *unconditional*, so the messages are injected
immediately when the declaration is encountered during execution.

You could create a single chain of those protocols like this:

.. code-block:: java

  Ports.protocol()
      .with(SomethingHappenedEvent.class)
          .trigger(new SomethingHappenedEvent());
      .with(FindDataRequest.class, FindDataResponse.class)
          .call(new FindDataRequest());


Combining Metaevents and Messages
=================================

Using the same event and request as above, we can combine metaevents and messages
like this:

.. code-block:: java

  Ports.protocol()
      .when(SomethingHappenedEvent.class)
          .triggers()
              .with(FindDataRequest.class, FindDataResponse.class)
                  .call(new FindDataRequest());

The ``with`` block for the ``FindDataRequest`` is now *conditional* and executes only
when the declared metaavent triggers. You can use all the syntax explained in the
previous sections to create arbitrary combinations of metaevents and messages.


Using Protocols in Testing
==========================

With protocols, you can create both unit tests and integration tests for
Ports components. In this context, a "unit test" involves just a single
component, while an "integration test" involves at least two components.

You can use protocols to both "mock away" the OUT ports that remain
unconnected when the components under tests are connected with each other and to
react to computational results, for example using assertions.

It is important to release all protocols before each individual test method
is executed. The only exception is when the protocols are valid for all tests,
however, that is rarely the case. For JUnit (version 5), you need a method like this:

.. code-block:: java

  @BeforeEach
  public void setup() {
      Ports.releaseProtocols();
  }

.. WARNING::
   If you do not release your protocols before each test, your tests will
   behave unpredictably and erratically. (Except when the protocols are valid
   for all tests.)

.. WARNING::
  Take care to place unconditional protocols *after* conditional
  protocols. This is because unconditional protocols
  are executed immediately when they are encountered and so cannot trigger
  conditional protocols that have not yet been declared.

In the following, you can have a look at a complete example, an integration
test of two simple components exchanging some numbers. In this test, we 
create a mock for a request.

We will use these event and request types:

.. code-block:: java

  @Response(Integer.class)
  public class IntRequest {

      private final int data;

      public IntRequest(int data) {
          this.data = data;
      }

      public int getData() {
          return data;
      }
  }


  @Response(Double.class)
  public class DoubleRequest {

      private final double data;

      public DoubleRequest(double data) {
         this.data = data;
      }

      public double getData() {
          return data;
      }
  }
  

  @Response(Double.class)
  @Response(String.class)
  public class EitherRequest {
  }

Further, we use these demo components:

.. code-block:: java

  public class FirstDemoComponent {

      @Out
      private Request<IntRequest, Integer> intRequest;

      @In
      private Double onDoubleRequest(DoubleRequest request) {
          int someIntegerFromOutside = intRequest.call(new IntRequest(1));
          return Math.sqrt(request.getData() + someIntegerFromOutside);
      }
  }


  public class SecondDemoComponent {

      @Out
      private Request<EitherRequest, Either<Double, String>> eitherRequest;

      @In
      private Integer onIntRequest(IntRequest request) {
          int eitherResponse = eitherRequest
                  .call(new EitherRequest())
                  .map(Double::intValue, string -> Integer.parseInt(string) / 2);

          return request.getData() + eitherResponse;
      }
  }


And this is the actual unit test (JUnit 5):

.. code-block:: java

  // We use this wrapper class in order to store values that are changeable
  // from within a closure, see below.
  static class ValueContainer<T> {

      public T value;

      public ValueContainer(T defaultValue) {
          value = defaultValue;
      }
  }

  @BeforeEach
  public void setup() {
      Ports.releaseProtocols();
  }

  @Test
  public void myTestMethod() {
      FirstDemoComponent firstDemoComponent = new FirstDemoComponent();
      SecondDemoComponent secondDemoComponent = new SecondDemoComponent();

      Ports.connect(firstDemoComponent).and(secondDemoComponent);

      Ports.register(firstDemoComponent, secondDemoComponent);

      // Protocols are based on lambdas which can only refer to final references.
      // Therefore, we use this simple ValueContainer in order to store computation results.
      ValueContainer<Double> result = new ValueContainer<>(Double.NaN);

      // Here come the conditional protocols:
      Ports.protocol()
          .when(EitherRequest.class, Double.class, String.class)
              .requests()
                  .respond(Either.b("4"))   // provide a mock response for EitherRequest
          .when(DoubleRequest.class, Double.class)
              .responds()
                  .do_(response -> result.value = response);   // store the result

      // This is an unconditional protocol:
      Ports.protocol()
          .with(DoubleRequest.class, Double.class)
              .call(new DoubleRequest(6.0));

      assertEquals(3.0, result.value);
  }


