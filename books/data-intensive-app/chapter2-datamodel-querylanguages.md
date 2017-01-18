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
    

    
