Lists and Non-Null
==================

Object types, scalars, and enums are the only kinds of types you can
define in Graphene. But when you use the types in other parts of the
schema, or in your query variable declarations, you can apply additional
type modifiers that affect validation of those values.

NonNull
-------

.. code:: python

    import graphene

    class Character(graphene.ObjectType):
        name = graphene.NonNull(graphene.String)


Here, we're using a ``String`` type and marking it as Non-Null by wrapping
it using the ``NonNull`` class. This means that our server always expects
to return a non-null value for this field, and if it ends up getting a
null value that will actually trigger a GraphQL execution error,
letting the client know that something has gone wrong.


The previous ``NonNull`` code snippet is also equivalent to:

.. code:: python

    import graphene

    class Character(graphene.ObjectType):
        name = graphene.String(required=True)


List
----

.. code:: python

    import graphene

    class Character(graphene.ObjectType):
        appears_in = graphene.List(graphene.String)

Lists work in a similar way: We can use a type modifier to mark a type as a
``List``, which indicates that this field will return a list of that type.
It works the same for arguments, where the validation step will expect a list
for that value.

A list can hold any type, including other ``ObjectType`` classes. The resolver
returns a plain Python iterable, and Graphene maps each item through the
declared type:

.. code:: python

    import graphene

    class Person(graphene.ObjectType):
        name = graphene.String()

    class Query(graphene.ObjectType):
        people = graphene.List(Person)

        def resolve_people(root, info):
            return [Person(name="Ada"), Person(name="Grace")]

This produces the type ``[Person]``, and the resolver's return value is
serialized item by item::

    {"data": {"people": [{"name": "Ada"}, {"name": "Grace"}]}}

The parent object needs to know nothing about the child type beyond the class
itself; ``Person`` only has to be defined before it is referenced.

Self-referencing lists
----------------------

A type that returns a list of itself would otherwise need to be defined before
it exists. Wrap the class in ``lambda`` so the reference is resolved lazily:

.. code:: python

    import graphene

    class User(graphene.ObjectType):
        name = graphene.String()
        friends = graphene.List(lambda: User)

        def resolve_friends(root, info):
            return [User(name=f"{root.name}'s friend")]

The same lazy form is what makes circular references between two types work.

NonNull Lists
-------------

By default items in a list will be considered nullable. To define a list without
any nullable items the type needs to be marked as ``NonNull``. For example:

.. code:: python

    import graphene

    class Character(graphene.ObjectType):
        appears_in = graphene.List(graphene.NonNull(graphene.String))

The above results in the type definition:

.. code::

    type Character {
        appearsIn: [String!]
    }

Combining ``NonNull`` with ``required=True`` makes the list itself non-null as
well as its items:

.. code:: python

    tags = graphene.List(graphene.NonNull(graphene.String), required=True)

which gives ``[String!]!`` — the list is always present and never contains
``null``.

