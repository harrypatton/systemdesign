Chapter 2: Data Models and Query Languages

#Introduction
1. Data Models - an important part of software.
2. Different on each layer
    * application level - object, specific to app.
    * storage - json, xml etc. general format
    * db layer - bytes (maybe)
    * hardware layer - electrical currents etc.
    
Basic idea: each layer hides the complexity of lower layer. This abstraction helps people focus on different levels.

This chapter discusses a range of general-purpose data models for ``data storage and queries``.

#Relational Model vs. Document Model

## Relational Model
1. Designed for business data processing
2. transaction processing like bank/stock/airline
3. batch processing like customer invoicing, payroll and reporting.

## the birth of NoSQL
Motivation

1. greater scalability
2. open source
3. specialized query operations that are not in relational models
4. more dynamic and expressive data model than relational schema

## The object-relational mismatch
* most apps are written in OOP. Requires an awkward translation layer between objects and database model (i.e., table, row and column).
* object-relational mapping (ORM) frameworks help but cannot completely hide the difference between the two models.

Linkedin example - a few ways
* traditional way - use a few tables profile, education and jobs to represent one person.
* later version of sql - a column can store multiple values.
* encoding all information as simple text like xml/json - app parses it.

It is a self-contained document, so json format is quite appropriate. Json has better locality than multiple table schema. It doesn't need multi-way join.

The one-to-many relationship seems like a tree. Json format is good at it.

### Many to one and many to many relationship
* for example like region id, it avoids duplicate data. Easy to update/maintain the underlying string.
* Document model doesn't support n:1 relationship well. Joining work goes from document database to application itself.
* data has a tendency of becoming more interconnected as features are added to applications. Links are better,
    * Organizations and schools changed from a few fields to entites. Each org and school has more information now.
    * Recommendation: it shows picture/name of people who write the recommendation on current user profile.
    
## History
Document-like DB showed up in history as hierarchical model; but no support for join and difficult for m:n mapping. Are we repeating history today?

Two models attempting to solve the problem,

### Network Model
* Hierarachical mode - one child has only one parent.
* Network model - the child can have multiple parents.
    * multiple access paths can reach the same node so developer needs to track. (hard)
 
With both the hierarchical and the network model, if you didn’t have a path to the data you wanted, you were in a difficult situation. 
    
### Relational Model
Everything is table. Query optimizer handles the complexity for apps.
    
## What changed in document DB
It uses hierarchical mode in one aspect: store nested records (i.e., one to many relationships) in parent record instead of separate table.

when handle many to one or many to many relationships, no fundamentally difference between document db and relational db. Both uses a reference (i.e., id) to refer to the real entity. It doesn't use network model.

Terms,

* ``Foreign key`` in relational DB
* ``document reference`` in document DB.
    
### document DB vs. Relational DB
data model is different.    
    
* document DB
    * for some applications it is closer to the data structures used by the application
    * schema flexibility
    * better performance due to locality
* relational model
    * better support for joins, many-to-one and many-to-many relationships.    
    
### which data model lead to simpler application code
* document DB
    * app uses document-like structure. ``shredding`` the document into multiple tables lead to cumbersome schemas and unnecessary complex application code.
    * poor support for joins in document db is not a problem if app doesn't need many-to-many relationships.
    
If the app needs many-to-many relationships, document db is not appearling,

* reduce the join by denormalization but it is hard to be consistent;
* app does multiple queries and join on app level, but it adds complexity and worse performance in app.
    
## Document DB - schema flexibility
it doesn't enforce schema. The code that reads the data assumes some kind of structure. A more accurate term is ``schema on reads``. Relational DB uses ``schema on write`` (DB enforces schema when write data.)
    
(I like this metaphor) Schema-on-read is similar to dynamic (run-time) type-checking in programming languages, whereas schema-on-write is similar to static (compile-time) type-checking.
    
The schema-on-read approach is advantageous if the data is heterogeneous, i.e. the items in the collection don’t all have the same structure for some reason, for example because:

* there are many different types of objects, and it is not practical to put each type of object in its own table, or
* the structure of the data is determined by external systems, over which you have no control, and which may change at any time.

In situations like these, a schema may hurt more than it helps, and schemaless documents can be a much more natural data model. But in cases where all records are expected to have the same structure, schemas are a useful mechanism for documenting and enforcing that structure.

## Document DB - data locality for queries
The document is usually stored as string. Read as a whole so it comes with better performance.

It recommends use fairly small size document because,

* reading a small part will read the whole document
* writing will write the whole document.
    
## Convergence of document and relational DBs
Some DBs added support for XML and Json. More will follow the path. Some document DBs are adding relational-like joins.

# Query languages for data














    
