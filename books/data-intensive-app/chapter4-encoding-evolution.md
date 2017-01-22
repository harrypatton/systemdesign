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
###Thrift and Protocol Buffers
