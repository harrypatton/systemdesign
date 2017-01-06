#Uber Service-Oriented Architecture
1. Monolithic architecture in the beginning - one option in one city. It is a clean way to solve core business problem at that time.
2. Things quickly change when expand to multiple cities and introduce more products. Hard to maintain the system,
	* everything tight coupled. hard to make encapsulation.
	* CI turns out to be a liability. Deployment has to deploy everything.
	* more dev work - more requests and features.
	* need tribal knowledge to make any change.

#Moving to SOA
1. Follow other companies to break down the monolith to multiple codebases to form SOA, i.e., microservices architecture.

#New Issues
Old issues are gone but new ones are coming.
* Obviousness
* Type Safety
* Resilience

##Obviousness
1. hard to find appropriate service among 500+ ones.
2. once found, not obvious how to use it.
3. a lot of other issues. In nutshell, they "converted legacy monolithic API to a distributed monolithic API."

It needs a standard way of communication to provide,

* type safety
* validation
* fault tolerance
* easy to provide client libraries
* cross language support
* tunable default timeouts and retry polices
* efficient testing and development

[Apache Thrift](http://thrift.apache.org/) is the solution to meet their needs best.

* datatypes and service interfaces are defined in a language agnostic file. 
* Then, code is generated to abstract the transport and encoding of RPC messages between services written in all of the languages we support (Python, Node, Go, etc.)

##Safety
It is type safety instead of security. Thrift guarantees safety by binding services to use strict contracts. The contract describes how to interact with that service including how to call service procedures, what inputs to provide, and what output to expect. 

##Resilience
Inspired by libraries from other companies and wrote their own.
