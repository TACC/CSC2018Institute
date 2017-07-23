# Authentication in HTTP and the Agave API

All the requests we made in the previous module were GET requests, and that's because the requests were all *anonymous* requests - we hadn't provided any identity information to prove who was making the requests. Typically, APIs will require a client to prove it is authorized to make changes to data (the POST, PUT and DELETE methods).

### Authentication
Authentication is the process of proving to a third-party (in our case, an API) you are who you say you are. At the DMV, this amounts to producing your driver's license. There are multiple ways of authenticating to APIs. We'll look at two of them.

#### HTTP Basic Auth
HTTP basic auth uses two pieces of information, a public identifier (such as a username) and a secret (such as a password). To authenticate to an API using HTTP Basic Auth, we create what an `Authorization` header.

<aside class="notice">
We ignored headers when we discussed HTTP, but every request and every response has them. Headers are attribute-value pairs that describe metadata about the message. Common headers include `Content-type` (will values such as `application/json` or `text/html` 
</aside>
  

