=======
History
=======

Changelog
=========

This changelog lists the most important improvements. Minor changes or fixes of
non-critical issues are not mentioned here.

* **0.6.0** --- (26 Apr 2021) introduced the ``@Pure`` annotation
* **0.5.11** --- (15 Mar 2021) added convenience methods, fixed exception streamlining in protocols
* **0.5.10** --- (19 Jan 2021) added convenience constructors
* **0.5.9** --- (10 Nov 2020) added ``Empty`` type, convenience constructors, fixed errors
* **0.5.8** --- (14 Oct 2020) fixed ``Either``/``Either3`` handling in protocols
* **0.5.7** --- (12 Oct 2020) added convenience constructors to ``Either``/``Either3``
* **0.5.6** --- (30 Sep 2020) added convenience constructors and convenience methods to ``Either``/``Either3``
* **0.5.5** --- (31 Aug 2020) added ``ofOptional`` to ``Either``, added ``reduce`` to tuples, fixed error in ``Triple``
* **0.5.4** --- (23 Jul 2020) fixed signature of ``andThen`` method of ``Either`` types
* **0.5.3** --- (10 Jul 2020) introduced component ownership in Vaadin-Spring module, improved ``Either`` API
* **0.5.2** --- (16 Jun 2020) request chaining
* **0.5.1** --- (02 Jun 2020) exception streamlining
* **0.5.0** --- (26 May 2020) protocols, asynchronicity & parallelism, type collection
* **0.4.1** --- (10 Apr 2020) syntax checking, union types
* **0.4.0** --- (04 Apr 2020) syntax checking, response type declarations
* **0.3.1** --- (24 May 2019) minor internal changes
* **0.3.0** --- (22 Feb 2019) various Vaadin/Spring improvements
* **0.2.4** --- (23 Nov 2018) first tagged release; Vaadin/Spring improvements
* **0.1.0** --- (25 Sep 2018) first release


Project History
===============

Despite still being in development, Ports already has a small but nice
success story:

I built the first version of Ports in late 2014/early 2015, back then in
C++ and under a different name ("Flow"; I later realized that this name had been
a bad choice). It was not a private project, but a means of rescuing a server
software that had become unstable and more or less unmaintainable due to
chaotic couplings and broken parallelism directly woven into the
business logic. I used this early variant
of the framework, whose main job was to control parallelism under the hood,
in order to refactor the software on the fly while features were still being
added. It
worked: after this operation, the server had become stable again, without
any negative performance impact.

I built the second version of the framework in ca. 2017 for a Python project
when I was working for the GALILEO satellite navigation project. This was a
very simple, rudimentary version that was used to build the main architecture
and to facilitate variant management. It worked, but it was a bit tedious
because of much boilerplate code that had to be written.

The Java version described in this document is the third incarnation, now
developed under my own auspices. The development started in 2018, and it is
used in several (still small) enterprise applications. I hope that
you will excuse any existing shortcomings. I do my best, but designing a new
framework sometimes requires trying something out --- and learning that it
was a bad idea. I hope that at the end of the day, more good than bad remains.

