#Uber TChannel
project page: https://github.com/uber/tchannel

* a protocol used for general RPC
* support
	* out-of-order response
	* high performance
	* intermediaries can make a forwarding decision quickly.
	* multiple languages

Doc: http://tchannel.readthedocs.io/en/latest/protocol/

##Problem Statement

TChannel was developed at Uber during a period of explosive growth in its SOA ecosystem. Services living in a large distributed system like this usually encounter similar classes of problems. Those problems often include:

* Service discovery -- how do I discover and consume the services around me?
* Fault tolerance -- how can I best insulate myself from failures outside of my control?
* Dapper-style tracing -- how can I identify and monitor bottlenecks across the entire system?
Design Goals

##Design Goals
TChannel aims to solve these problems by providing a protocol for clients and servers with an intelligent routing mesh (referred to as Hyperbahn) connecting the two.

Consider those SOA problems again:

* Service discovery: All producers and consumers register themselves with the routing mesh. Consumers access producers only by their name; no need to know about hosts or ports.
* Fault tolerance: The routing mesh tracks things like failure rates and SLA violations. It can intelligently detect unhealthy hosts and remove them from the pool of available hosts.
* Request tracing: Built into the protocol as a first class citizen.

Consolidating logic between producers and consumers also allows us to push core features to our entire SOA without requiring applications to update libraries.

Additionally, when developing this protocol and routing mesh we had the following goals in mind:

* The protocol must be easy to implement in multiple languages, especially Javascript, Python, and Go. We do not want to restrict ourselves to one language.
* Async behavior is fundamental. We follow a traditional request / response model with out of order responses; slow requests must not block subsequent, faster requests at the head of the line.
* Large requests/responses may/must be broken into fragments to be sent progressively (i.e., "streaming" requests are supported).
* TChannel can be used with arbitrary serialization schemes, e.g. JSON and Thrift, although Thrift is preferred for its IDL, type-safety, and validation guarantees.
* The routing mesh needs a high-performance forwarding path. Intermediaries must be able to make a forwarding decision quickly.
* Optional checksums for request/response integrity.

##Message flow
1. bi-directional request/response protocol.
2. each party is equal.
3. Possible to have multiple connections between the same pair of peers.
4. One message corresponds to all frames that share the same message id. Note, it seems that a message consists of multiple frames.

