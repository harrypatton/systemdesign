#Encoding and Evolution

Application always change. Feature changes require data changes in most cases.

* Relational databases - although there is schema change, there is only one schema at any one point.
* Schemaless databases - may contain both old and new data format at a time.

No matter server-side applications or client-side applications, they have to support both old and new data schema. It is easy to understand for client applications. For server ones, think of rolling upgrade - we don't upgrade all nodes at once.

In other words, old and new code, old and new data could coexist at the same time. We need to maintain compatibility in ``both directions``.
* backward compatibility: new code can handle old data. In general, it is easy to handle.
* forward compatibility: old code can handle new data. it could be tricky.

The chapter will check a few common data formats (JSON, XML etc.),
* how they handle compatibility
* how they are used for data storage
* how they are used for communication, e.g., web service, rest, rpc, message queues

#Formats for Encoding Data
Code works with data in two representations,
* objects (in memory). array, hash table, trees, struct etc. Optimized for CPU.
* byte sequences. passed to other process/network or on disk. 

The translation from in-memory to byte sequence is called ``encoding`` (aka, ``serialization``, ``marshalling``).  The reverse translation is called ``decoding`` (aka, ``deserialization``, ``parsing``, ``unmarshalling``)

Too many encoding formats are available.

##Language-specific formats
A lot of built-in support in languages.

Advantages
* Convenient
* Easy to save and restore

Disadvantages
* Specific to the language so other languages cannot read.
* Decoding process need to instantiate arbitrary classes - security concern.
* Versioning data is often an afterthought. ([me]: I'm not sure.)
* Efficiency is also often an afterthought. ([me]: I'm not sure)

conclusion: **not recommended**.

##JSON, XML and binary variants
JSON, XML, CVS: supported by many languages, well known and supported.
* XML - too verbose and unnecessary complicated.
* JSON - popular due to built-in support by web browsers; simpler.
* CSV - less powerful.

Subtle problems
* ``ambiguity around number encoding``. CSV and XML cannot tell string and number. JSON cannot tell int and floating point (thus no precision). E.g., a number larger than 2^53 is used to identify each tweet. JSON returned by Twitter API has to provide ID twice - one as number and the other as decimal.
* Good support for unicode strings, but ``no support for binary strings``. workaround: use Base64 to encode binary as text.
* Schema support is optional. XSD is commonly used for XML, but not many JSON-based tools using schemas. It replies on apps to encode/decode data correctly.
* CSV - no schema.

Despite these flaws, JSON/XML/CSV are **good enough for many purposes**.

###Binary Encoding
JSON and XML use a lot of space compared to binary format. It leads to binary editions for JSON and XML. Some extend data type and other keep data model unchanged.

###Thrift and Protocol Buffers
[me] I like the two formats. 

Thrift originates from Facebook and Protocol Buffers originates from Google.
* Both require a schema for any data to be encoded using IDL.
* Code generation tool can produce classes for different languages.

Thrift
* Binary protocol
* Compact Protocol - it uses variable-length integer, and other changes.

Protocol Buffers - similar to compact protocol in Thrift.

All can have required and optional properties.

#### Field Tags and Schema Evolution
Encoded data doesn't have any information about field name. It only has field tag (i.e., property id). 

As Id, field tag is very important to value meaning,
* Cannot change field tag.
* Adding a new field is ok - old code ignores it and new code can handle it, **except** we cannot add a new ``required`` field though; otherwise new code cannot read old data which doesn't new field.
* Removing a field is ok only if it is ``optional``. Note, the same tag number cannot be used again in the future.

#### Data Types and Schema Evolution
Sometimes it is ok to change data type. It may lose precision or get truncated. E.g, change 32-bit int to 64 bit int. New code can handle old data, but old code will truncate new data (using 64 bit int).

### Avro
Apache Avro ws a sub-project of Hadoop because Thrift didn't fit the scenario.

It has two schema languages,
* IDL for human editing
* one based on JSON for machine-readable.

Encoded data contains values only. It has length and then UTF-8 bytes. To parse the data, it has to check the schema. Coding reading and writing the data have to use the same schema.

## Writer's Schema and Reader's Schema
Definitions
* When app writes data, it needs a schema to encode the data. it is called ``writer's schema``.
* When app read data, it needs a schema to decode the data. it is called ``reader's schema``.

The key idea with Avro is ``both writer's and reader's schemas don't have to be the same`` - they only ``need to be compatible``.

When decode code, Avro library checks both reader and writer schemas to resolve conflicts.
* order doesn't matter. It uses field name.
* when a field exists on writer but not reader, reader just ignores it.
* when a field exists on reader but not writer, reader will uses ``default`` value defined in reader schema.

 **Question**: how does the library know the writer schema? The data itself doesn't embed schema content. Actually library caller will pass in writer schema or a schema id which can be read from database to get full schema content. A few examples,
 * it is designed for data analytics - scan a ton of rows at once. A writer schema is probably defined in the beginning of file.
 * every encoded data can include a version number in the beginning.
* communicate schema between processes.

version number is good to have. It could be incremental integer or a hash of schema content.

## Schema Evolution Rules
Forward compatibility: code with ``old version of reader schema`` can read data written in ``new writer schema``.
Backward compatibility:  ``new version of reader schema`` can read data written in ``old writer schema``.

To maintain compatibility, add or remove a field that has a default value.
* when add a field without a default value, code with new version of reader schema cannot read data written in old version.
* when remove a field without a default value, code with old version of reader schema cannot read data written in new version.

## Dynamically Generated Schemas
One advantage over Thrift and Protocol Buffers - no tag number in schema.

``dynamically generated schema`` - dump db tables into Avro. It has encoded data and also the schema in which column name becomes field name. when db table schema changes, it just generates a new version of schema. Runtime just check if compatible by field name. If we use Thrift or Protocol Buffers, each column needs a tag number assigned manually; admin need to maintain the tag number; don't reassign a tag used before.

[me] to be frank, the difference is only between tag id and field name. Tag id is a little bit harder to manage in this case but not terrible.

## Code generation and dynamically typed language
Code generation is good for static typed languages such as Jave, C#, C++. It is not good for dynamically typed programming languages like JavaScript or Python because there is no compile-time type checker.

Avro can be used with and w/o code generation.

##The merits of schemas
Schema is used to describe a ``binary`` encoding format.

Many data systems implement some proprietary binary encoding based on schemas. DB uses the schema to send queries to DB and get back responses.

* much more compact than various ``binary JSON`` variants because of field name omitting.
* serve as valuable form of doc. It must be up-to-date.
* check forward/backward compatibility by saving all schemas.
* support for statically typed programming languages.

schema evolution provides the same flexibility as schemaless or schema-on-read JSON database, and also provide better guarantees about your data and tooling.

#Modes of Data Flow

##Data flow through databases
* write to db => encode
* read from db => decode.
* backward compatibility is required: newer code can read data.
* forward compatibility: required. a few processes can access at the same time. Some are old and the other are new code.

 Application level change is probably required to take care of data loss issues.

###Different values written at different times
It is easy to upgrade all applications at once, but hard to migrate all data to new schema at once in DB, so DB needs to allow simple schema changes.

###Archival storage
common to take a snapshot of db for backup.

##Data flow through services: REST and RPC
client <-> server. The API exposed by server is known as ``service``.

*web: http is used as the transport protocol. API implemented on top is application specific, so the client and server need to agree on APIs.
* ``Service-oriented architecture`` (aka ``microservices architecture``): decompose a large application into smaller services by area of functionality.
* individual service can evolve by itself. so expect both old and new versions of service and clients running. encoding data must be compatible. (note from me: instead of considering compatibility, a service can provide a different API for new/updated feature.).

### Web Services
when http is used as underlying protocol for talking to service, this is called ``web service``.  User cases,
* client app (e.g., mobile device) makes request to service over HTTP.
* one service calls another service in same organization.
* one service calls another service in different organization (usually via internet).

Two popular approaches, ``REST`` and ``SOAP``.

REST design philosophy
* simple data format
* using url to identifying resources
* using http features for cache control, authentication, content negotiation.

SOAP
* XML-based protocol to make network API request.
* Independent on http. try to avoid using http features. It has own standards.

### Remote Procedure Calls (RPC)
RPC: making a request to remote network service as if it calls functions in the same local process. 

Network request is very different from local call,
* result is unpredictable. 
* may not return result
* retrying cause the same action to run multiple times (because response is lost).
* Performance/response varies a lot.
* need to encode parameter value. complicated for large objects.
* data type probably incompatible between client and server.

conclusion: hard to treat it as a local object. 

### RPC future
Custom RPC protocols with binary encoding format has better performance than JSON + REST.

* REST - good for public APIs.
* RPC - good for internal services.

### Data Encoding and Evolution for RPC
Service is updated first and then client, so backward compatibility on request, and forward compatibility on responses. In other words, service needs to handle old-version request, and client can handle new-version response.

Service may not have control on client side, so it has to maintain compatibility for a long time or infinity.  If a compatibility-breaking change is required, it has to provide multiple versions SxS.

##Message passing data flow
client sends message -> message broker (message queue) -> received by service.
* act as a buffer if service is too busy
* redeliver message if previous service crashed
* decouple client and server. (client doesn't know the server)
* one message can be sent to multiple receivers.

Difference from RPC: no response to client.

###Message Brokers
Open source - RabbitMQ, ActiveMQ etc.

Easy to understand.

###Distributed Actor Framework
``Action Model``: a programming model for concurrency in a single process. Instead of handling threads directly, it encapsulates logic in ``actors``. 
* Actors use async message for communication among them. 
* The framework schedules the work for actors.
* Message may get lost.

distributed actor framework = message broker + actor programming model

#Summary
recommended to read the section to refresh what we read. It is a long chapter:).
