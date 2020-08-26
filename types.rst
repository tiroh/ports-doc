=========================
The Ports Type Collection
=========================

Ports provides a collection of convenience types that make handling of requests,
commands, and errors easier. They don't have any specific ties to message handling,
so they can also serve as general-purpose types.

The following examples show only the most common use cases. The respective classes
contain much more methods that come in handy in different situations.


Either, Either3
===============

``Either`` and ``Either3`` are union types that represent exactly one of either two
(``Either``) or three (``Either3``) possible values.

``Either`` and ``Either3`` provide type-safe handling of different kinds of state
without having to use error-prone ``if-else`` constructs. In addition, they can
provide a simpler syntax for certain kinds of computations. Their constructors do
not accept ``null`` values.

Of course, you can build arbitrary union types using only ``Either``. ``Either3`` is just
a convenience type.

.. code-block:: java

  // Declare an Either that represents either an Integer or a String:
  Either<Integer, String> either;
  
  // Let this Either represent an Integer using the "Either.a" constructor:
  either = Either.a(37);
  
  // Handle the two cases:
  either.on(
      integer -> { System.out.println("The integer is " + integer); },
      string -> { System.out.println("There is a string: " + string); }
  );
  
  // Do a transformation:
  double result = either.map(
      integer -> integer * 1.5,
      string -> Double.NaN
  );


Nothing, Unknown
================

The ``Nothing`` type indicates the absence of a meaningful value, while the ``Unknown``
type indicates that a meaningful value is not known, but might still exist.

The difference between these types is subtle, but reasonable. Let's have a look at
an example.

Let's assume we are designing a software for configuring laptop computers. A laptop
might have a discrete graphics card. When we request information about the discrete
graphics card, there are two scenarios that we would like to differentiate:

* There is no graphics card. In this case, the service responsible would return ``Nothing``.
* For some reason, we are not able to retrieve the required information. In this
  case, the service responsible would return ``Unknown``.

You can use these types in conjunction with ``Either`` or ``Either3``:

.. code-block:: java

  Either3<Data, Nothing, Failure> response = request.call(new Request(...));
  
  response.on(
      data -> { ... },
      nothing -> { ... },
      failure -> { ... }
  );

``Nothing`` cannot be instantiated. Instead, you can use the static ``Nothing.INSTANCE``
member.


Success, Failure
================

``Success`` and ``Failure`` are types that do not represent values but the procedural
outcome of an operation. Like ``Nothing``, they are particularly helpful in conjunction
with ``Either`` and ``Either3`` in the context of requests and commands:

.. code-block:: java

  Either<Success, Failure> status = command.call(new Command(...));
  
  status.on(
      success -> { ... },
      failure -> { ... }
  );

Also, if you have a command that cannot fail, you can define its response simply to be a
``Success`` in order to indicate this fact.

``Success`` cannot be instantiated. Instead, you can use the static ``Success.INSTANCE``
member. ``Failure`` provides both a constructor and a static default instance ``Failure.INSTANCE``.

The ``Failure`` type is used for streamlining exceptions. Please see :doc:`requests`
and :doc:`exception-handling` for more information.


Pair, PairX, Triple, TripleX
============================

The four types ``Pair``, ``PairX``, ``Triple``, and ``TripleX`` represent tuples. They
all implement Ports's ``Tuple`` interface.

``Pair`` and ``Triple`` represent pairs and triples containing values of arbitrary
types, while ``PairX`` and ``TripleX`` represent pairs and triples containing only values of one type.

The tuple types provide many convenience methods and interoperability with ``Either`` and
``Either3``. These are only a few examples:

.. code-block:: java

  Pair<Integer, String> pair = Tuple.of(1, "two");
  PairX<Float> floatPair = Tuple.ofX(1.0f, 2.0f);
  
  pair.onNotNull(
      integer -> { ... },
      string -> { ... }
  );
  
  PairX<Double> doublePair = floatPair.map(floatValue -> floatValue * 2.0);
  
  Pair<Either<Integer, Nothing>, Either<String, Nothing>> eithers = pair.toEithers();
  Pair<Optional<Integer>, Optional<String>> optionals = pair.toOptionals();
  
  TripleX<Float> triple = Tuple.ofX(1.0f, 2.0f, 3.0f);
  boolean isContained = triple.containsDistinct(floatPair);
  PairX<Float> pairBC = triple.pairBC();


Container
=========

The ``Container`` class provides a simple wrapper around an arbitrary type ``T``. This type
exists only because Java lambdas cannot form closures with non-final local variables, making state
modifications of those variables difficult when working with closures. Most of the time,
this is not a problem because you don't want lambdas to modify state anyway, but
there are exceptions to this general rule, mostly for technical reasons.

For example, when you are writing tests using protocols, you may want your
lambdas to modify some variable in order to indicate to JUnit a certain test outcome.

You instantiate a ``Container`` using its ``of`` constructor. You can access its value
directly via its public ``value`` member:

.. code-block:: java

  Container<Integer> intContainer = Container.of(37);
  
  intContainer.value *= 2;


TriFunction
===========

The ``TriFunction`` interface works similar to the well-known ``Function`` and
``BiFunction`` interfaces of the ``java.util.function`` package. The difference is
that while ``Function``
and ``BiFunction`` only support functions with 1 or 2 parameters, respectively,
the ``TriFunction`` interface supports 3 parameters.
