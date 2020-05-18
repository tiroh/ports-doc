======================
Theoretical Background
======================


Componential Abstraction
========================

In software development, there are different kinds of abstraction --- e.g.,
data abstraction or functional abstraction. The Ports Framework employs a third type of
abstraction that I call *componential abstraction*. Componential abstraction is a
way to abstract from functions.

Let ``f`` and ``g`` be functions. Let's assume that they can be composed like
this: ``f(g())``. So, they are composable, which is nice, but unfortunately,
they are neither independently reusable nor independently deployable units
of software. Thus, they only support programming in the small, but not
programming in the large.

So, instead of using ``f`` and ``g`` directly, we apply to them a *componential
abstraction operator* and retrieve components
``F`` and ``G``. The signatures of ``f`` and ``g`` have now become IN ports of
``F`` and ``G``, respectively. The advantage of componential abstraction is that
it makes ``f`` and ``g`` composable, reusable, and deployable, all at the same
time.


Componential Normal Form
========================

A software system is in *componential normal form* if it has applied
componential abstraction thoroughly. Hence, its only executable units are components
that are composable, reusable, and deployable.


Why Should Components Only Depend on Data Types, not Functions?
===============================================================

Because depending on functions would violate the componential normal form.
For example, if we passed around functions freely and called them whereever
and however we like, this would betray the concept of
abstracting from functions by means of componentization. The three goals of
composability, reusability, and deployability would not be achieved.

Whenever you feel that a component needs a function, remember that what it really
needs is data of a certain type. In this case, you should apply componential
abstraction and create two things: **(a)**
an OUT port that requests the required data, **(b)** a component that represents the
function you were thinking about. This preserves the componential normal form of the
system and, by means of componential abstraction, provides the team with a reusable function.

Of course, in order to support maintainability, components can subsume several
functions at once. It is not in each and every case necessary or desirable to
have an individual component for each function.


What is the Problem with Dependency Injection?
==============================================

Naturally, the question arises why all of this shouldn't be achievable with dependency
injection. Well: technically speaking, it is, and if you have to use DI-based frameworks like
Spring, by all means please do your best! But it won't be easy, because there is a
problem on the conceptual level.

Dependency injection in its typical interpretation (e.g., Spring) is based on type polymorphism,
meaning that the "dependencies" are expressed using polymorphic interface
types, and the "injection" deals with interface implementations. Let's call this kind of
dependency injection "*interface-based dependency injection*" from now on.

So, interface-based dependency injection does not lead to dependency on data types but
to dependency on interfaces. Interfaces represent components. Our hope is that we will
decouple those components from each other, but unfortunately, the opposite is
the case. Assume that there are two components **A** and **B** that are application-specific,
that is, they don't originate from a library or framework but belong to the solution.
Component **A** depends on an interface **IF** that is implemented by
component **B**. Then:

* *de facto*, component **A** cannot be used without component **B** because there is rarely
  an alternative provider of **IF**,
* that means that in turn the dependencies of **B** must be satisfied,
* even if there is an alternative provider **C**, it is very likely not a drop-in replacement for **B** because
  it might violate functional and temporal couplings between **A** and **B** that have evolved over time,
* component **A** does not necessarily require the complete functionality of **IF**, yet it
  is forced to import it completely, thus suggesting to the developer to make use of it
  and form a coupling,
* whenever **A** has a new requirement, developers are *very* tempted to implement
  it within existing interfaces instead of creating new ones, thus creating
  more and more couplings,
* component **A** cannot be tested without (a) someone implementing **IF** and (b) taking care
  that this mock implementation mimicks the behavior of **B**,
* the interface of **B** cannot be changed without breaking **A** (and all the other dependants),
* B cannot be changed internally without breaking **A** (and all the other dependants)
  because of functional and temporal couplings between **A** and **B** that have evolved over time,
* probably even **A** cannot be changed internally without breaking **B** for the same reason,
* and of course: oftentimes, developers not even make the effort of creating interfaces but just
  inject the implementations directly. This way, the DI framework is reduced to mere
  syntactic sugar without any design benefit, but why bother?

You may object that some of these items are not the consequence of interface-based dependency
injection but the result of bad development practice --- in other words, bad code.
Well --- yes, strictly speaking, that is true, but part of the problem is that
interface-based dependency injection fosters bad development practice and
makes bad code a very tempting, easy choice.

So, dependency injection in its typical interpretation means that *de facto*,
there is no hinge between **A** and **B** --- the two are glued together just as if
there were no inversion of control at all. And of course, a real-world system does not consist
of only two components but of dozens or hundreds --- the consequence is a big, sticky network
of couplings, held together by interfaces and interface-based dependency injection.
Now, this explains why I consider this kind of dependency injection a deep design flaw.

If a component is a house, then an interface dependency represents a "broken window",
an entry point for "intruders" --- that is, couplings.

A component --- which should be, mind you, an independent agent! --- has no
intrinsic interest in interfaces. Interfaces are imposed on it in an
arbitrary way by the system designer. In reality, what an agent is interested in
is data --- or *data types*, to be exact.

The Ports Framework tries to alleviate this problem by avoiding the notion of an
"interface type". Instead, it emphasizes the notion of the *data type*.
Dependencies and contracts are expressed only on the
level of individual data types. This bears the potential for greatly improving
maintainability.


Won't This Cause New Problems?
==============================

Problems can indeed arise by the introduction of a distributed, event-oriented
architecture. Such architectures can quickly lead to issues with performance and
understandability, thus leading to maintainability problems.

Firstly, we should realize that typical enterprise applications (using Spring or similar
frameworks) *are* distributed systems already; if not technically, then at least
conceptually. Whoever implements enterprise software is already
dealing with this type of system and has to understand its rules of engagement anyway.
The Ports Framework or the concepts promoted in this document are independent
of this fact.

Secondly, these problems can be avoided if the  body of knowledge regarding
this kind of design is respected. Distributed systems are not a new
invention, and much knowledge and experience is readily available.


