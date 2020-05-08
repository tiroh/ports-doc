===============
Core Principles
===============

#. **A sender never makes an assumption about who is the receiver of its messages.**
#. **A sender never makes an assumption about what the receivers will do (except for
   command messages).**
#. **A receiver never makes an assumption about who is the sender of incoming messages.**
#. **A component only sends or receives messages that make sense in the context of its
   own concern.** In other words, a component cannot reasonably care for something
   that's not part of its own concern; if it does, that's a breach of encapsulation.
   There are design patterns for translating between
   concerns, the most prominent one being model-view-controller where the controller
   has the job of arbitrating between the concerns of the view and the model.
#. **Each agent is responsible for handling its own errors.** An agent must never
   delegate error handling to somebody else. An exception must only be sent (but not thrown,
   see below!) in the case of an undefined state where comprehensive handling is
   not possible because of missing information. See also :doc:`exception-handling`
   for more details.
#. **A component must never throw an exception beyond its component boundary.** Using
   exceptions within a component is fine, but they must not leave the component's
   boundary, both for technical reasons and design reasons. Error or status
   information should be communicated in the form of either exception events or
   requests/commands that return an ``Either``. See :doc:`exception-handling` for
   more details.
#. **A component's ports can only be used by that component itself, not by other components.**
   For example, it would be a pretty twisted idea to create a getter for an OUT port and
   to pass that OUT port around. While technically possible, that would
   be a severe abuse of this possibility. Instead, create a design where each
   agent is truly independent and does not fall victim to foreign intrusion.
#. **Components should only send and return thread- and process-independent data, ideally only
   data-transfer objects without logic.** No resources, no files,
   no streams, no threads, and, in particular, no functions. Of course, on a
   technical level, functions can be *encoded*
   as data (everything can), but that does not *make* them data on the conceptual level.
   See also the next principle for another issue with functions.
#. **Components should only depend on datatypes, not functions.**
   Otherwise, the componentization would not be in
   normal form. Please read :doc:`background` for more details on this.
