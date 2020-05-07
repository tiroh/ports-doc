========================
The Vaadin-Spring Tandem
========================

Keep in mind the following items:

#. The Ports Framework manages pushing data from the backend to the UI automatically.
   You don't have to dig into the Vaadin documentation and copy-paste their excellent
   code examples. When you have an event port in the backend (singleton scope) that is
   connected to some component in the UI scope, you can just trigger the event. The
   framework will detect this crossing of scope boundaries and take care that the
   push takes place.
#. In principle, Ports works with Vaadin's push mode set to both AUTOMATIC and MANUAL
   (both work in the tests).
   In practice, in some applications only MANUAL works. It is likely that this is a
   configuration issue on the side of the applications, but for now, it is recommended
   to set the push mode to MANUAL.
#. If you use MANUAL push mode, you unfortunately have to call
   ``PortsPushMode.setManual()`` as early as possible in your application lifecycle.
   This will change in the next release (``PortsPushMode`` will be removed).
#. In principle, only use the singleton scope and the UI scope. If you need a UI
   component that can be instantiated multiple times per UI, use the prototype scope.
   Singleton scope does not require any annotation.
#. Spring only instantiates components that are either in the singleton scope or that are referenced.
   Because Ports avoids interface-based dependency injection, some UI components are not
   referenced (in particular, presenters) and so are not instantiated. Those components
   have to be manually referenced
   so that Spring instantiates them (using ``ApplicationContext.getBean``).
#. You should prefer the ``@SpringComponent`` and ``@Service`` annotations for your components.
   Spring's ``@Component`` annotation collides with Vaadin's ``Component`` class.
#. The difference between a "component" and a "service" is that a "service" provides some
   sort of business-logic service to others, usually in the form of requests or commands, while a
   "component" either is a UI element or has a housekeeping role of some sort. Of course,
   this is only a conceptual
   distinction. Technically, there is no difference between "components" and "services".
