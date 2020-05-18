==============
Why use Ports?
==============


For a more in-depth discussion about the concept behind Ports and why this concept
has significant advantages when compared to typical (Spring-like) dependency injection, please
read :doc:`background`.


Advantages
==========

* Ports helps you achieving better maintainability by replacing interface-based
  dependency injection with data-type-based dependency injection. This way, it is much
  easier for you to keep components decoupled.
* Ports manages asynchronicity and parallelism automatically. Business code can be
  kept cleen of any manual synchronization and threading directives.
* Ports makes implementing *variants* of your product a little bit less of a pain.
  (Variant management is always a pain, but the way Ports works provides a certain
  kind of analgesic.)
* Integration tests become much easier to design and implement.

Of course, like each and every framework and methodology, Ports
does not make all problems and challenges magically go away. You still have to
know and follow basic software design rules and
best practices. The part of Ports in this is that it paves the way in front of you
(*if and only if* you embrace its design principles).


Disadvantages
=============

* Ports is mainly made for representing the static aspects of your software
  architecture. Ports can also handle dynamic parts, but there are
  use cases where it reaches its wits' end, at least in its current stage of
  development. In these cases, you still have to use classic method calls
  instead of ports communication.
* Ports introduces its own philosophy of how component interaction should be
  managed on a technical level. This can cause conflict with other frameworks
  like Spring that rely on classic dependency injection. Ports is compatible
  with Spring via its ``ports-vaadinspring`` module (currently only in
  conjunction with Vaadin), but you may encounter some rough edges that require
  a (simple) workaround.

Roadmap
=======

The following major features are planned for future releases:

* A proprietary IOC container so that Spring is not required anymore.
* Routing of events and requests.
* Automatic extraction of graphical design documentation directly from your
  source code. (No need for maintaining UML component diagrams.)
* Someday, support for more platforms and programming languages (e.g., Kotlin,
  C++, C#).
