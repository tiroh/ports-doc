====================
The Types Collection
====================

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


Nothing
=======

The ``Nothing`` type indicates the absence of a meaningful value. You can use this type
in conjunction with an ``Either`` or ``Either3`` to indicate that a value is missing:

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