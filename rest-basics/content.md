## What is REST?
**REST** (REpresentational State Transfer) is an increasingly popular method of creating, returning, updating and deleting data, particularly from a program that should have limited access to the server.

## Why is REST used?
REST is a technique, not a technology, so it isn’t something you need to make your project work.
REST is just a set of guidelines for creating an API for a web service.
These guidelines will help you design a simple and effective API.

If you don’t need to provide a web service in your project, you may not have a use for REST.

### What is a Web Service?
A **web service** is an agent for your server, allowing other software (**clients**) to talk to it easily and securely.
Without web services, a client may need to know complex and sensitive information about a system, such as database structure and passwords; which is unacceptable in most cases.
Web services allow the server to maintain control at all times, while providing an easy way for clients to communicate with it.

Web services work by providing an API, which typically allows clients to create, return, update or delete data from a server. APIs can be public or private and are often designed so that clients aren’t limited by one computer language or platform.

## Advantages of REST over alternative API models
The most common alternative to REST is SOAP, which uses it’s own standard to wrap up requests and responses with XML. As a result it can be chunky and fairly complex to design for. It seems that SOAP overcomplicates web services.

By contrast:

- REST is lightweight and fast - It’s just a resource path and data. No wrapper or standards are needed.
- REST is simple to understand and use - The simplicity of REST helps in designing an intuitive API. It also makes it easy to create or adapt clients.
- REST is powerful - Anything you can do with HTTP or other protocols, you can do with REST.
- REST is flexible - There are no limits, just guidelines. For example data can be returned in any format. It is up to the designer.
- REST is scalable - Your API will not likely need to be rethought when it needs to support new functionality.


## How does REST work?
This article will focus on using REST with the **HTTP protocol**, although others are sometimes used.
You will therefore need a basic understanding of HTTP.

REST takes advantage of **HTTP verbs**, particularly GET, POST, PUT and DELETE. 
You will have used GET and POST methods when creating web forms, but REST is more particular on when and how each method should be used.

To clarify, your API can use these HTTP methods in any way it wants to. But it can only be considered **RESTful** if it follows the following guidelines:

Method | Use | Safe?[^1] | Idempotent?[^2]
--- | --- | --- | ---
GET | Returning a resource | Yes | Yes
POST | Creating a new resource (without supplying a unique identifier) | No | No
PUT | Creating or updating a full resource (with a specific unique identifier) | No | Yes
PATCH | Partially update a resource | No | Yes
DELETE | Deleting a resource | No | Yes

[^1]: a method is considered *safe* if it could never alter data on the server, as opposed to *unsafe* which could.
[^2]: *idempotent* means if the request was repeated the data on the server wouldn’t change.


## Example of how REST might be used
Let’s pretend you want to write a film rating application to share with friends.
You create the database on your web server.
Each of your friends wants to create their own interface so you decide to write a RESTful API for them.
The API documentation you give to your friends might look like this:

To get the whole library of rated films
~~~~
GET https://myserver/films
~~~~

To get information about the film with an ID of `jaws`
~~~~
GET https://myserver/films/jaws
~~~~

To add a new film to the library
~~~~
POST https://myserver/films
~~~~
~~~~ json
{ "title": "Star Wars: The Force Awakens", "rating": 5 }
~~~~

To update all information about ‘jaws’
~~~~
PUT https://myserver/films/jaws
~~~~
~~~~ json
{ "title": "Jaws", "rating": 3 }
~~~~

To partially update ‘Jaws’
~~~~
PATCH https://myserver/films/jaws
~~~~
~~~~ json
{ "rating": 4 }
~~~~

To delete ‘jaws’ from the film library
~~~~
DELETE https://myserver/films/jaws
~~~~


REST is all about **resources**, so it’s important to use nouns in the URI, as opposed to verbs such as ‘add’ or ‘update’ etc. The request type (GET, PUT etc) defines what to do with resource.  

When extra data is needed in a request, consider sending it as a querystring, or as JSON or another data-serialisation format.

E.g. to specify which fields you want to return:
~~~~
GET https://myserver/films/jaws?fields=director,year,rating
~~~~


## How could I write the API?
First you should consider configuring your web server to convert friendly URLs into something more meaningful for your programming environment.

For example - your API might be a single PHP file at `https://myserver/film-api.php`. You could then configure your web server to rewrite RESTful URLs to all point to film-api.php.
So `https://myserver/films/jaws` could be translated to `https://myserver/film-api.php/films/jaws`. It should be fairly simple then to parse the resources path in your API script.

If your API is fairly complex, it may be worth building it with a framework such as [Laravel](http://laravel.com/) (or it's lighter counterpart, [Lumen](http://lumen.laravel.com/), as this will make routing and database queries easier and more structured.


### Tip - Rewriting URLs
If you are using Apache web server, then look no further than [mod_rewrite](https://24ways.org/2013/url-rewriting-for-the-fearful/)

The API script would then need to determine the request method (GET, PUT etc) and use any data with the request to query the database and return a proper response.

### Tip - Getting the Request Method
To get the request method in PHP, use the `$_SERVER['REQUEST_METHOD']` global variable.


## How should the API format data to return?
This is entirely up to you, but as the data needs to be processed by the client, it should be a popular data-serialisation format such as [JSON](http://http://json.org/).

Crucially, the HTTP response that an API returns will have to be properly formed. That means setting the `Content-Type`, `Status` and other important headers. Remember you are just returning HTTP headers and raw data.

### Note - Caching
RESTful APIs should define show long to cache responses for.

To suggest caching the response for a specified period, use `Cache-Control: max-age=3600`, where 3600 is the number of seconds the response should be cached for.
To suggest not caching the response, use the header `Cache-Control: no-cache`

### Tip - Error handling
Often the HTTP Status codes don’t provide enough detail to properly explain what happened. Consider sending error details in the response data and explaining your errors in the documentation.


## How could I write a client?
To write a client that can use the web service, you can use any computer language that can make HTTP requests. For example JavaScript can make requests using AJAX[^3], and Ruby has good native support. PHP users tend to prefer to use the [cURL](http://curl.haxx.se/) extension or a framework to make HTTP requests as it is a lot simpler than the language’s native methods.

[^3]: Some web browsers unfortunately only support GET and POST.


## How should clients format data to send?
When you submit an HTML form, the input data is usually serialised (formatted as one long string). The MIME type for this format is `application/x-www-form-urlencoded`. However if your client supports it, you can use different formats for your data as long as you specify the **Content-Type** as part of the request. JSON (`application/json`) is a popular alternative. Of course, the server must be able to understand and correctly handle the data format.


## How do I make the API secure?
In terms of secure data transfer, using SSL is essential. To switch from HTTP to HTTPS, you only need to purchase a SSL certificate from your hosting company. You should also consider throwing an error to the client if they use HTTP as it compromises their security.

Remember, it is up to the server whether or not to action API requests. This can be done with HTTP authentication, or requiring credentials as part of each request.

If you don’t want the client to have to send credentials on each request, then the server could generate a random authentication token when the client ‘logs in’. The client then just needs to provide this token for new requests. The server has complete control of whether or not the token is valid, so you could design it so that the token expires after a set time.

Truly RESTful APIs are **stateless** and should not keep sessions or any other data for a client.


## TL;DR
- REST is a set of guidelines on how to design an API.
- REST calls are usually just HTTP requests.
- REST requests describe resources (books, films etc), with the type of request defining what to do with the resource (GET, POST, PUT, DELETE).
- RESTful APIs are simple, portable, scalable and stateless.
- REST responses should always include a Status code and explicitly encourage or discourage caching results.


## Useful resources
- [List of HTTP headers](http://en.wikipedia.org/wiki/List_of_HTTP_header_fields)
- [List of HTTP status codes](http://en.wikipedia.org/wiki/List_of_HTTP_status_codes)
- [REST best practices](http://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api)


*[API]: Application Programming Interface
*[SOAP]: Simple Object Access Protocol
