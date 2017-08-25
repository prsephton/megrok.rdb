megrok.rdb
==========

Introduction
------------

The `megrok.rdb` package adds powerful relational database support to
Grok, based on the powerful [SQLAlchemy] library. It makes available a
new `megrok.rdb.Model` and `megrok.rdb.Container` which behave much like
ones in core Grok, but are instead backed by a relational database.

In this document we will show you how to use `megrok.rdb`.

Declarative models
------------------

`megrok.rdb` uses SQLAlchemy’s ORM system, in particular its declarative
extension, almost directly. `megrok.rdb` just supplies a few special
base classes and directives to make things easier, and a few other
conveniences that help with integration with Grok.

We first import the SQLAlchemy bits we’ll need later:

    >>> from sqlalchemy import Column, ForeignKey
    >>> from sqlalchemy.types import Integer, String
    >>> from sqlalchemy.orm import relation

SQLAlchemy groups database schema information into a unit called
`MetaData`. The schema can be reflected from the database schema, or can
be created from a schema defined in Python. With `megrok.rdb` we
typically do the latter, from within the content classes that they are
mapped to using the ORM. We need to have some metadata to associate our
content classes with.

Let’s set up the metadata object:

    >>> from megrok import rdb
    >>> metadata = rdb.MetaData()

Now we’ll set up a few content classes. We’ll have a very simple
structure where a (university) department has zero or more courses
associated with it. First we’ll define a container that can contain
courses:

    >>> class Courses(rdb.Container):
    ...    pass

That’s all. If the `rdb.key` directive is not used the key in the
container will be defined as the (possibly automatically assigned)
primary key in the database.

FIXME a hack to make things work in doctests. In some particular setup
this hack wasn’t needed anymore, but I am unable at this time to
reestablish this combination of packages:

    >>> __file__ = 'foo'

Now we can set up the `Department` class. This has the `courses`
relation that links to its courses:

    >>> class Department(rdb.Model):
    ...   rdb.metadata(metadata)
    ...
    ...   id = Column('id', Integer, primary_key=True)
    ...   name = Column('name', String(50))
    ... 
    ...   courses = relation('Course', 
    ...                       backref='department',
    ...                       collection_class=Courses)

This is very similar to the way you’d use `sqlalchemy.ext.declarative`,
but there are a few differences:

    * we inherit from ``rdb.Model`` to make this behave like a Grok model.

-   We don’t need to use `__tablename__` to set up the table name. By
    default the table name will be the class name, lowercased, but you
    can override this by using the `rdb.tablename` directive.
-   we need to make explicit the metadata object that is used. We do
    this in the tests, though in Grok applications it’s enough to use

  [SQLAlchemy]: http://www.sqlalchemy.org
