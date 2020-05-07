==============
Best Practices
==============

General Design
==============

#. **An ideal Ports agent has no dependencies.** In other words, it has a nullary
   constructor, no setters, and no dependency injection from Spring.
#. **Using setters or dependency injection from Spring is only allowed for technical
   reasons, not for design reasons.** There are times when we just need to use
   setters or Spring's DI because of technical reasons, and that's fine. However,
   we must understand that this is only a technical workaround and not good design practice.
#. **When an undefined state is entered, no assumptions can be made about anything.**
   This is simply because the state is *undefined* -- it's in the name. In particular,
   you cannot assume that a certain exception already has been handled or will be
   handled. When an undefined state is entered, all you can do is try to *contain the
   damage* and hope for the best. (If you could do more than that, the state wouldn't
   be undefined and you should definitely do more.) This strategy must be used by all
   agents that are stakeholders in the failed operation.
#. **Differentiate between data getters and data finders.** It is useful to realize
   that there is a conceptual difference between *getting* data and *finding* data. When
   you issue a request that *gets* data, you are saying that the data must exist, and
   if it doesn't, that means that the database is corrupt (the system has entered undefined state).
   On the other hand, when you issue a request that *finds* data, it is ok when nothing
   is found -- the result set is just empty, which does not indicate undefined state.
   This difference is fundamental and can lead to a plethora of both bugs and maintainability
   problems if not clearly understood. It is best if you name your requests such that
   it is clear whether they are getters or finders.
#. **Do not mix the technical domain with the solution domain.** The technical domain
   is the domain of your customer, while the solution domain is the domain of your
   software product. It is very important to understand that they are *unrelated* and must
   remain so. It is tempting to model the solution domain closely after the image
   of the technical domain. However, this is a very fateful coupling. There are several
   reasons: **(a)** the technical domain is constantly changing, even in those cases
   where the customer tells you that nothing will ever change (the customer has usually
   no idea how complex his own domain is; in addition, there may be third-party
   stakeholders who have the power to induce change); **(b)** the technical domain almost always
   has a broken design with countless deep flaws and rough edges and a long,
   complicated history. If you model your solution
   according to this design, you will transfer all those flaws into your software product,
   which is a completely unnecessary mistake that will trouble your solution for years
   to come. You should always create a clean design and then arbitrate between the
   technical domain and your solution domain.
#. **In the technical domain, there are no getters. There are only finders.** Getters
   are a specialty of the solution domain because they require correctness
   guarantees that only the solution domain can provide. Even if the technical
   domain specifies explicit contracts of existence, uniqueness etc., these contracts
   cannot be made contracts of the solution domain because they are not
   controlled by the solution domain. See also the previous item.
   

Databases
=========

The following best practices are essential for maintainable, decoupled database code:

#. The backend is not allowed to send entity instances to the frontend, and the frontend
   is not allowed to reference entity types. Entity data must always be wrapped in DTOs
   (data transfer objects). Also, backend services shall not exchange entity instances
   amongst each other.
#. DTO types must always be specific to exactly one service. That is, service ``X`` must never
   create instances of DTOs that belong to another service ``Y``.
#. When you implement a DTO for an existing table ``X``, name its class ``XDto`` and
   provide it with a member ``id`` that contains the unique identifier (solution domain)
   of the record.
#. When a DTO contains a property ``P`` of another entity ``Y``, name the DTO member ``yP``.
#. DTOs do not need to correspond to a physical database table. They can also be used for
   aggregated data.
#. You can never use data from the technical domain as unique identifiers (this follows
   from the best practices above).
#. Don't use the term "id" for data from the technical domain. It is best if the term "id"
   is immediately recognizable as a unique identifier from the solution domain. In the
   technical domain, use the term "reference".
#. Implement the distinction between getters and finders (see above).

