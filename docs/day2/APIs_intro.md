# Introduction to Application Programming Interfaces (APIs)

In general an Application Programming Interface (API) establishes the protocols and methods for one piece of a program to communicate with another.
  * Allow larger software systems to be built from smaller components.
  * Allow the same component/code to be used by different systems.
  * Insulate consumers from changes to the implementation.
  
Some examples of APIs:
  * In OOP languages, abstract classes provide the interface for all concrete classes to implement.
  * Software libraries provide an external interface for consuming programs.
  * Web APIs (or “web services”) provide interfaces for computer programs to communicate over the internet.
  
### Web APIs
In this course, we will focus on web APIs (or HTTP APIs). These are interfaces that are exposed over HTTP.

#### HTTP - the Protocol of the Internet

HTTP is how two computers on the internet communicate with each other. In particular, our web browsers use HTTP when communicating with web servers running web applications.

The basics of the protocol are:
  * web resources are identified with URLs (Uniform Resource Locators). Initially, `resources` were just files on directories on the server, but today resources refer to more general objects.
  * HTTP "verbs" represent actions to take on the resource. The most common verbs are GET, POST, PUT and DELETE.
  * A request is made up of a URL, a HTTP verb, and a message.
  * A response consists of a status code (numerical between 100-599) and a message. The first digit of the status code specifies the kind of response: 
    * 1xx - informational
    * 2xx - success
    * 3xx - redirection
    * 4xx - error in the request (client)
    * 5xx - error fulfilling a valid request (server).

#### REST APIs
REST (Representational state transfer) is a way of building APIs for computer programs on the internet leveraging HTTP in following sense:
  * A program on computer 1 interacts with a program on computer 2 by making an HTTP request to it. 
  * “Resources” are the nouns of the application domain and are associated with URLs.
    * The API has a base URL from which all other URLs in the API are formed. For example, `https://api.github.com`
    * The other URLs in the API are either:
      * entire collections, such as `<base_url>/users`, `<base_url>/files`, `<base_url>/programs`, etc.
      * specific items in a collection, such as `<base_url>/users/3` or `<base_url>/files/test.txt`
  * “Operations” are the actions that can be taken on the resources and are associated with HTTP verbs:
    * GET - list items in a collection or retrieve a specific item in the collection.
    * POST - create a new item in the collection based on the description in the message.
    * PUT - replace an item in a collection with the description in the message.
    * DELETE - delete an item in a collection.
  * Response messages often make use of some data serialization format standard such as JSON or XML.
  
Open a web browser and navigate to: `https://api.github.com`
  * Your browser made a GET request to the github API. What you see are a list of attribute-value pairs that describe the other URLs in the API.
  
```
Exercise 1. What URL would we use to get a list of github users?

Exercise 2. Can you find a URL returning the details of your github account?

Exercise 3. Navigate to the URL that returns your followers.
```


### Using the Python requests library

We'll use the python requests library to interact with the github API programmatically. Open up a Jupyter notebook and follow along.

In order to do anything, we need to:
```
import requests
```

The basic usage of the requests library is as follows:
```
# make a request
response = requests.<method>(url=some_url, data=some_message, <other options>)

# work with the response:

response.status_code -- the status code

response.content -- the raw content

response.json() -- for services returning JSON, create a Python list or dictionary from the response message.
```


Let's explore the github API using the requests library in a Jupyter notebook. Along the way, let's be sure to write code snippets in the notebook to:
```
1. retrieve a list of github users.
2. retrieve details about a particular user.
3. get the list of followers for a particular user
4. given two github accounts, determine which account was:
   4a. created first
   4b. updated most recently
   4c. determine if they have any common "starred" repositories
```
