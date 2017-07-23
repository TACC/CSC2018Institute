# Introduction to Authentication in HTTP

All the requests we made in the previous module were GET requests, and that's because the requests were all *anonymous* requests - we hadn't provided any identity information to prove who was making the requests. Typically, APIs will require a client to prove it is authorized to make changes to data (the POST, PUT and DELETE methods).

> We will be illustrating some of the concepts using the github API. To follow along you will need a github account. If you do not have a github account already, we encourage you to get one. They are free and are a great way to engage the open source community. Sign up here: https://github.com/join?source=header-home


## Authentication
Authentication is the process of proving to a third-party (in our case, an API) you are who you say you are. At the DMV, this amounts to producing your driver's license. There are multiple ways of authenticating to APIs. We'll look at two of them.

### HTTP Basic Auth
HTTP basic auth uses two pieces of information, a public identifier (such as a username) and a secret (such as a password). To authenticate to an API using HTTP Basic Auth, we create an `Authorization` header.

> We ignored headers when we discussed HTTP, but each request and response has them. Headers are attribute-value pairs that describe metadata about the message. Common headers include `Content-type` (with values such as `application/json` or `text/html` describing the type of content in the message) `Cookie` (used to store session data), and `Authorization` (for authentication).

To use HTTP Basic Auth, we have to create a string combing the identifier and the secret with a colon and base64 encoding it. Fortunately, the `requests` library makes it easy on us: simply pass `auth=(identifier, secret)` into the request.

```
Exercise 1. Make an authenticated request to the github API using HTTP Basic Auth with your username and password. Use the `https://api.github.com/user` to pull information about the authenticated user.
```

JSON is a very popular data serialization format for HTTP APIs, and github's API uses it. JSON uses attribute-value pairs to describe data, and values can be one of a standard set of types including: Number, String, Boolean, Array (i.e., list), or Object (nested collection of attribute-value pairs).

To tell github to create new object for us, we make a POST request passing JSON describing the object we want to create. The `requests` library also makes this easy: we simply create a python dictionary with the description of our object and pass it to the request like `json=obj`.

```
Exercise 2. Let's create a new github repository by making a POST request using Basic Auth to the `https://api.github.com/user/repos` URL. Make sure to git the repository a description. Consult the github API documentation (https://developer.github.com/v3/repos/#create) to determine how to create your json description.

Exercise 3. Retrieve a complete description of the repository you just created using an authenticated GET request to the repo's `url` (not it's html_url).
```

The github API uses the PATCH method for updating to an object, though it seems to require a full description of the object (all required fields). 

```
Exercise 4. Let's use a PATCH request to turn off the "projects" feature of our test repo. Verify that your change took. Consult the github docs as needed: https://developer.github.com/v3/repos/#edit
```

Finally, since this was just an exercise, let's remove this test repository.

```
Exercise 5. Use a DELETE request to remove the test repository we created. API docs: https://developer.github.com/v3/repos/#delete-a-repository
```

> Note that it is typical for RESTful APIs to return an empty response to a DELETE request. What happens if you try to call the `json()` method on the github response to request in exercise 5? How can you validate that the request was succesful?

At this point the repository should be deleted. 

Hopefully the power of HTTP APIs is starting to become clear. Say we needed to create 100 repositories? Or 1000? That would be a lot of clicks in the github website, but using Python, we could write a short program to create them in no time, assuming we knew the names of the repositories we needed to create. Moreover, we used Python because everyone knows it's the best programming language (jk), but nothing about what we did required Python. HTTP is a language-agnostic protocol: any programming language capable of sending messages over a socket can be used, and virtually every modern language has a high-level HTTP library such as `requests` for handling many of the minute details for us.

### OAuth

HTTP Basic Auth is pretty easy to use, but it has some limitations, the primary one being that we have to pass the secret into every request. At best this is annoying and a little insecure, but what if we are writing an application for others and they do not want to share their github password with us? This is where OAuth comes in. 

If you ever used a site that said "Log in with your Facebook or Google account" then you have used OAuth. We will explore OAuth in more detail in the next module.
