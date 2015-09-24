---
title:  Tomcat Hates Encoded Slashes
author: rnicholus
categories: Tomcat server HTTP REST Java JavaScript
excerpt: "After sending a GET request containing escaped slashes to our Tomcat-hosted endpoint, we expect to receive a 2xx response. But instead, our server responds with a 404. What happened?"
---

Let's picture a small segment of a typical web application in order to better understand this problem. We'll focus mostly on the server here, but the client plays a key role as well. On the server, we have a REST endpoint that exists to proxy information from _another_ endpoint. This handler accepts GET requests. The _other_ endpoint to proxy is included as a path parameter. The signature of our endpoint handler looks something like this:

```java
@GET
@Path("proxy/{url}")
String getData(@PathParam("url") String url);
```

In case it is not already obvious, the above code is part of a Java interface, and the annotations are part of the [javax.ws.rs package][javaxws], which is a collection of interfaces and annotations that align with the [JSR 311 specification][jsr311] maintained by the [Java Community Process group][jcp].

The exact implementation of the above endpoint method is unimportant, so I'll omit it for the sake of brevity. When called, it will simply make a GET request to the address specified at the end of the `@Path`, which is stored in the `url` parameter. The response from the resource associated with this `url` will return some data in the response, and this data will then be returned to whichever client called our "proxy/" endpoint. Our client code may very well be running in a browser, in which case it may look something like this:

```javascript
var resourceToProxy =
  encodeURIComponent('http://widen.com/careers')

fetch('/proxy/' + resourceToProxy)
  .then(function(response) {
    if (response.ok) {
      return response.text()
    }
    else {
      var error = new Error(response.statusText)
      error.response = response
      throw error
    }
  })
  .then(function(proxiedData) {
    // handle proxied data in response
  })
  .catch(function(err) {
    console.error(error)
  })
```

The above client-side code utilizes the native [Fetch API][fetch], but you could accomplish the same call with [`XMLHttpRequest`][xhr]. In fact, the client doesn't even need to be browser-based - it could very well live on a server, coded in any language under the sun.


## The problem

After executing the above request, we would fully expect to receive a 2xx response from our server and then handle the proxied data via the success function of fetch's returned promise. But instead, our error handler is invoked. Looking closer, we see that our server returned a response code of [404][404]. But we've clearly defined our endpoint, and even verified that our request is being properly set to this endpoint. So what happened?


## The cause

I left out one _minor_ detail - our web server is [Apache Tomcat][tomcat], which is very common when Java is the primary server language. In version 6.0.10 (released around February of 2007), [the Tomcat team patched a security hole][tomcat-security]. This involved treating encoded forward and backslashes in the URL as path delimiters. So, our URI of "/proxy/http%3A%2F%2Fwiden.com%2Fcareers" is being expanded to "/proxy/http%3A//widen.com/careers" _before_ it is routed to a matching endpoint handler. Of course, this endpoint is not accounted for, and our server rejects the request as a result.


## The solution

There are two ways to work around this behavior. The first involves adjusting a Java system property on our application server. Setting the `org.apache.tomcat.util.buf.UDecoder.ALLOW_ENCODED_SLASH` system property to `true` should allow our request to go through as expected. The default value of this property is `false`. Another option involves switching our GET request to a POST and including the url to proxy in the message-body. This is arguably the safest option. If we take this route, our server endpoint will look like this:

```java
@POST
@Path("proxy/url")
String getData(String url);
```

The body of our POST request will be made available to our handler method via the `url` parameter. We will have to make a slight adjustment to our client-side code as well:

```javascript
var resourceToProxy = 'http://widen.com/careers'
fetch('/proxy/url', {method: 'POST', body: resourceToProxy})
  .then(function(response) {
    if (response.ok) {
      return response.text()
    }
    else {
      var error = new Error(response.statusText)
      error.response = response
      throw error
    }
  })
  .then(function(proxiedData) {
    // handle proxied data in response
  })
  .catch(function(err) {
    console.error(error)
  })
```

Note that there is no need to encode the proxy endpoint address anymore, since it is no longer part of the request URI. The changes to our JavaScript are limited to the first two lines of code. This will result in A POST request to our "/proxy/url" endpoint with a Content-Type of "text/plain". As expected, the body of our request will contain the `resourceToProxy` value. After making these changes, everything works as intended, and we are able to successfully proxy a third-party endpoint through our server.


[404]: http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.4.5
[fetch]: http://davidwalsh.name/fetch
[javaxws]: https://docs.oracle.com/javaee/7/api/javax/ws/rs/package-summary.html
[jcp]: https://www.jcp.org/en/home/index
[jsr311]: https://jcp.org/en/jsr/detail?id=311
[tomcat]: http://tomcat.apache.org/
[tomcat-security]: http://tomcat.apache.org/security-6.html#Fixed_in_Apache_Tomcat_6.0.10
[xhr]: https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest
