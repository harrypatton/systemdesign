* Stateless: more like a function which always return the same result for the same argument. It doesn't store any state for future calling. Good example is restful service.
* Stateful: it stores internal value for future calling. Good example is winform desktop app. Asp.net ViewState is another example.
* A stateful app is one that stores information about what has happened or changed since it started running. Any public info about what "mode" it is in, or how many records is has processed, or whatever, makes it stateful.
* Stateless apps don't expose any of that information. They give the same response to the same request, function or method call, every time. HTTP is stateless in its raw form - if you do a GET to a particular URL, you get (theoretically) the same response every time. The exception of course is when we start adding statefulness on top, e.g. with ASP.NET web apps :) But if you think of a static website with only HTML files and images, you'll know what I mean.

##References
* http://stackoverflow.com/questions/5329618/stateless-vs-stateful-i-could-use-some-concrete-information
* https://www.quora.com/What-is-stateless-and-statefull-web-architecture
