============
Event Causes
============

Events are great for attaching additional logic to game actions, but
they have the drawback of providing next to no context as to what has **caused** that event to
occur. The ``Cause`` object allows providing and receiving additional contextual information about
the event. This contextual information can then used to modify the behavior of your event listener.

For example, a world protection plugin needs information on what player has caused a
ChangeBlockEvent to occur before they can decide if the event should be cancelled or not.
Rather than go with the traditional route of creating a multitude of subevents for the different source
conditions this information is instead provided in the ``Cause`` of the event.

Every event provides a ``Cause`` object which can be interogated
for the information pertaining to why the event was fired. The Cause object can be retrived from
an event by simply calling ``event.getCause()``.

Retrieving objects from a Cause
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Structurally, a ``Cause`` object contains a sequential list of objects. There are several methods of
retrieving information from a Cause object which we will discuss here, for a more complete
listing please see the javadocs **link**.

.. note::

    The objects within a cause are ordered such that the first object is the most immediate
    cause of the event, and subsequent objects are of decreasing importance and/or may only
    provide contextual information.

``Cause#root()`` returns the first object within the cause. This object is the most immediate or direct cause of the
event. Since a ``Cause`` may not be empty, it is guaranteed to have a ``root``.

``Cause#first(Class)`` returns the first object in the cause chain whose type is either the
same as or is a subtype of the given class. For example given a cause which contained a player
followed by an entity ``[Player, Entity, ...]``

.. code-block:: java

    @Listener
    public void onEvent(ExampleCauseEvent event) {
        Cause cause = event.getCause(); // [Player, Entity]
        Optional<Player> firstPlayer = cause.first(Player.class); // 1
        Optionml<Entity> firstEntity = cause.first(Entity.class); // 2
    }

Both optionals would contain the player object as it's type directly matched request for a
Player type and it matched the request for an Entity type as Player is a subtype of Entity.

``Cause#last(Class)`` is similar to *Cause#first(Class)* except it returns the last value in
the cause chain matching the type.

Continuing from the example above, if we instead changed it to call ``Cause.last(Class)`` the first
optional would contain the player object still, but the second optional would now contain
the entity that we passed in the second position of the cause.

``Cause.has(Class)`` returns a boolean value and can be used to check if a cause chain
contains any object matching the provided type.

``Cause.all()`` simply returns all objects within the cause allowing more advanced handling.

Named Causes
~~~~~~~~~~~~

Sometimes the ordering of objects within the cause isn't enough to get the proper idea of what
an object represents in relation to the event. This is where ``NamedCause`` comes in. Named
causes provide a method for tagging objects within a cause with a **unique** name allowing them
to be easily identified and requested. Some examples of use cases for named causes is the
`Notifier` of a GrowBlockEvent or the `Source` of a DamageEntityEvent.

**Retrieving a named entry from a cause**

.. code-block:: java

    @Listener
    public void onGrow(GrowBlockEvent event) {
        Optional<Player> notifier = event.getCause().get(NamedCause.NOTIFIER);
    }

This example makes use of ``Cause.get(String name)`` which can be used to retrive the object
associated with a name if it is present within the cause chain. Additionally
``Cause.getNamedCauses()`` provides a ``Map<String, Object>`` which can be used to find all
present names and their associated objects.

.. note::

    Some common identifying names for ``NamedCause``\ s are present as static fields in the
    ``NamedCause`` class. Identifiers which are specific to certain events can often be found
    as static fields on the event class, for example ``DamageEntityEvent.SOURCE``.

Creating custom Causes
~~~~~~~~~~~~~~~~~~~~~~

Creating a cause to use when firing an event is extremely easy. The hardest part is deciding
what information to include in the cause. If you're firing an event from your plugin which is
usually triggered through other means perhaps you want to include your plugin container so
other plugins know that the event comes from your plugin. Or if you are firing the event on
behalf of a player due to some action it's usually a good idea to include that player in
the cause.

.. note::

    Cause objects are immutable therefore cannot be modified once created.

Using ``Cause.of(Object, Object...)`` or ``Cause.ofNullable(Object, Object...)`` you can construct a cause
from a series of objects. The objects will be added to the cause chain in the order that they
are passed to the method, so the first object parameter will become the root cause.
Remember that a ``Cause`` may not be empty, so at least one non-null parameter is always required.

If you already have a cause object and would like to append some more objects to the
chain you can use ``Cause.with(Object...)``. This constructs a new Cause object containing
first the objects already present in the original cause, then followed by the additional
objects that you provided.

Finally if you wish to add an object to a cause with a defined named first call
``NamedCause.of(String name, Object object)`` and then pass the returned ``NamedCause`` instance
to the cause chain as you would a normal object. Simply calling ``Cause.of(name, object)``
will result in a cause chain containing two objects (the String name, and then the object),
rather than one object with an attached name.
