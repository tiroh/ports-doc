=======
History
=======

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

