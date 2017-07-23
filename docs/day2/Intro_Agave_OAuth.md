# Introduction to OAuth and the Agave API
In this module we introduce OAuth and the Agave API.

### OAuth
HTTP Basic Auth is pretty easy to use, but it has some limitations, the primary one being that we have to pass the secret into every request. At best this is annoying and a little insecure, but what if we are writing an application for others and they do not want to share their github password with us? This is where OAuth comes in. 

#### Basic concepts
  The key concepts in OAuth are:
  * Three parties are involved in OAuth: a provider, a client and a user. 
    * An OAuth provider represents an authoritative source of identity information (essentially, any entity that maintains a collection of user accounts).
    * An OAuth client typically represents an *application* such as a web or mobile application, a command line script, etc.
    * Clients use the OAuth provider to interact with resources on behalf of the user.
  * OAuth provides different protocols called *grant types* to enable clients to use different processes to gain access to a user's resources.
  * A successful OAuth authentication results in the client obtaining a short term *access token* representing the user from the provider. The client can then use this access token to access resources until the token expires. It is typical for OAuth tokens to expire in a few hours, though there are mechanisms for "refreshing" them.


If you ever used a site that said "Log in with your Facebook or Google account" then you have used OAuth. An in-depth treatment of OAuth is beyond the scope of this course, but we will see it in a little detail when we explore the Agave API.


### Agave API

Agave (https://agaveapi.co/) is an API platform developed here at TACC for conducting computational science experiments on HPC and cloud resources. 

<center><img src="../resources/agave.png" style="height:300px;"></center>

We will introduce Agave towards two objectives:
  * Introduce OAuth at a basic level
  * Use the Agave cloud storage API (i.e., files API) to store and retrieve files we want to keep "long term".

> In fact, Agave has many more APIs that may be of interest to you such as the Jobs API for launching and managing jobs on remote computers such as Stampede, or the Meta API for storing arbitrary metadata objects about scientific experiments. 

Agave is a full-featured OAuth provider and leverages the TACC identity system behind the scenes. The hands on portions below will leverage your TACC username and password.

To get started, we are going to generate a set of Agave client keys (OAuth client credentials). Generating OAuth clients uses HTTP Basic Authentication with your TACC username and password. We will do this in a Jupyter notebook. The 
  * The Agave base URL is `https://api.tacc.utexas.edu`
  * The URL for the Agave clients service is `https://api.tacc.utexas.edu/clients/v2`; the `v2` here indicates that we are using version 2 of the Agave platform (the current version). All core Agave API URLs are versioned.
  * 


```
Exercise. Use HTTP Basic Auth with your TACC username and password against the Agave clients service to list all your OAuth clients. Explore some of the endpoints within the clients service. Discuss the notion of API subscriptions.
```

#### Creating a client
To create a simple OAuth client that has access to all basic Agave APIs, we need to make a POST request to the clients service. The only required field we need to pass in is `clientName` to give a name to our client. Each client we create must have a unique name.

In order to create an Agave OAuth client, we make a POST request to the Agave clients service. The only required field we need to pass in is `clientName` to give a name to our client. Each client we create must have a unique name.

```
Exercise. Generate an Agave client by making a POST request to the clients service.
```

Clients are identified by their key and secret. These two important properties are returned in the fields `consumerKey` and `consumerSecret` respectively. 

```
Exercise. Extract the key and secret from the client you just generated into variables in the notebook. We will need these fields to interact with the Agave OAuth token service and generate an access token.
```

### Generating Access and Refresh Tokens

We're now ready to generate an OAuth token. This token will represent both the user and client application. In this case, they are owned by the same individual, but in general they will not be.

To generate an OAuth token, we make a POST request to Agave's token service. We have to pass in several fields to make the request:
  * `username` - the username of the end user represented by the token
  * `password` - the password of the end user represented by the token
  * `grant_type` - the OAuth grant type we are using (in this case `password`).
  * `scope` - the OAuth scope we want. In this case, simply use `PRODUCTION`.

> It might seem odd that we are passing in the username and password when that was one of the things we were trying to avoid. There are two points to make: 
  * first, we only have to pass it in once to get the access token and then all subsequent requests will use the access token.
  * second, in general we could use a different grant type to not have to collect the password at all, but in this introduction we are taking the simplest approach.

A note about scopes:
> scopes in OAuth can be used to limit/restrict the kinds of access the client has. The scope of `PRODUCTION` indicates that the client will have full access (i.e., read, write, update, execute) to the user's resources.

Also, keep in mind that the Agave `token` service URL is simply `https://api.tacc.utexas.edu/token` - it does not have a `v2` in it.
> Semantically, the idea was that the token service was independent of the Agave platform version.

The response, if successful, will contain the following fields:
  * `access_token` - the access token representing the end user and client.
  * `refresh_token` - a token that can be used to retrieve a new access token using the `refresh_token` grant type.
  * `expires_in` - the time, in seconds, that the `access_token` will expire.
  * `token_type` - the token type (will have value `bearer` -- this is to comply with the OAuth spec).
  * `scope` - the scope associated with the token (will have value `default`)
  
```
Exercise. Generate an access and refresh token using the Agave token endpoint.
```

### Using an access token
Once we have an access token we are ready to interact with the rest of the Agave services. All requests to Agave using this access token will be done on behalf of the user whose credentials were used to retrieve the token (as well as the OAuth client that was used).

In order to make a request to Agave using the access token, we need to pass the token into the Authorization header of the request. The value of the header must be formatted like so: "Bearer <access_token>"


As a simple check, we'll use the Agave Profiles service to pull the "profile" associated with this token. The Agave Profiles service maintains some details about registered users. 
  * Base URL for the Agave Profiles service: https://api.tacc.utexas.edu/profiles/v2
  * "me" endpoint - https://api.tacc.utexas.edu/profiles/v2/me ("me" is a special reserved word in Agave to indicate we want information about the associated token.)

```
Exercise. Use the access token to make a request to the Agave profiles service "me" endpoint to retrieve the profile associated with the token.
```


