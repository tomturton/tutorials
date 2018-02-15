The following are some miscellaneous REST notes that I didn't want to clutter up my [introductory REST tutorial](/posts/rest-basics).


# Related 'resources'

It is often useful to manage resources that are related to resources.
For example, getting a list of reviews for a film.

We could provide a filter to the `reviews` resource collection, like so:

~~~~
GET /reviews?film=Jaws
~~~~

However this is ugly and not particularly intuitive.

I think, a better way might be:

~~~
GET /films/Jaws/reviews
~~~

We might also want to access other resources belonging to this film:

~~~
GET /films/Jaws/cast
GET /films/Jaws/crew
GET /films/Jaws/locations
GET /films/Jaws/quotes
~~~

Despite *cast*, *crew* and *locations* not belonging to any one *film*, I still believe this is the most sensible way to request these resources.


# Security

Security is paramount, for both your system and your users. The first step is to encrypt the connection between the server and your clients, which can be made possible by installing an SSL certificate.

SSL has been getting a lot of press at the moment. Google are starting to [discriminate against non-SSL sites](https://googlewebmastercentral.blogspot.co.uk/2014/08/https-as-ranking-signal.html) and the industry is starting to [address demand for free and simple SSL certificates](https://letsencrypt.org).
If your website or API only serves public information, then you may not feel the need to encrypt connections.
However, using `https://` not only looks good, but helps your visitors to trust the content hasn't been manipulated before it gets to them (known as a man-in-the-middle attack).

Authenticating a user for a website has always been a bit of a breeze. You force them to log in, and then create a session or cookie to keep that user logged in while they browse the site.
However REST is explicitly *stateless*, meaning there are no sessions. Every call to a RESTful API should be treated as unique, and must be authenticated.


## Basic authentication

There are a few ways to approach this. We could use employ [Basic authentication](https://en.wikipedia.org/wiki/Basic_access_authentication), so that for every request a user needs to include their username and password.
It's a quick and easy solution, but isn't ideal for a number of reasons:

- The client has to send credentials for every request.
- Only *username* and *password* fields are supported, which may not be suitable for your application (e.g. if you use two-factor authentication).
- Some clients may not securely store user credentials.
- Users have no way of deauthorising devices (e.g. if their phone is stolen).


## Tokens

An increasingly common way of getting round most of these problems is by authenticating a user once, and then providing them with a security token for future requests (sent as a custom HTTP header).

The beauty of this method is that your server can easily manage the validity of each token.
You might want tokens to expire after a set period, forcing the user to reauthenticate every day.
Also, while the user could be logged in to multiple clients simultaneously, they could deauthorise any one of them if necessary.


## OAuth

Perhaps the safest way of authorising your users is to let one of the big players do it for you.

With [OAuth](https://en.wikipedia.org/wiki/OAuth), if your users have account with Google, Facebook, Twitter, GitHub or [another major player](https://en.wikipedia.org/wiki/List_of_OAuth_providers), then you can let your users log in with one of these organisations.

You would have to trust organisations are able to vouch for your users, but the chances are they can do this a lot better than you anyway.


# Versioning

It is important to think about API versioning from the very beginning, as APIs usually grow and change, and you don't want to catch anyone out if they are using an old version of an API.

I like having version number as part of the request URL. It is clear and avoids extra request headers:

~~~
GET https://myserver/film-api/1.0/films
~~~

You should also decide on a versioning pattern from the start, and stick to it. I like [semantic versioning](http://semver.org/).


# Filtering, sorting, and paginating

If you have anything bigger than a small data collection, it is likely you will want to add support for filtering, sorting, and paginating a collection of resources.
Besides being a useful feature, adding filters to your SQL queries will keep you API fast.


## Filtering

Simply use a query parameter for each of the fields you would allow filtering on:

~~~
GET /films?year=1975
GET /films?director=Steven+Spielberg
GET /films?year=1975&genre=thriller
~~~


## Sorting

I use a *sort* parameter with comma separated values to denote the field priority to sort by:

~~~
GET /films?sort=director
GET /films?sort=year,genre,director
~~~

You will also want to support sorting in descending order. I use a `-` character to specify such fields:

~~~
GET /films?sort=-year,director
~~~


## Pagination

Even when filtering, it is still often more efficient and useful to only return a handful of results.
I use a `page` parameter to determine which page of results to return:

~~~
GET /films?page=2
~~~

I could also allow the client to specify how many results to return per page.
I might override this if the number is too high, so not to choke the server.

~~~
GET /films?rows=30&page=3
~~~

# HATEOAS

Pagination brings me nicely onto HATEOAS; a fairly controversial part of REST, that specifies APIs need to include meta information about the response.

Using pagination as an example, this could include a total record count and URIs to other pages.

I like to group meta information and the actual data seperately:

~~~ json
{
    "meta": {
        "per_page": 25,
        "current_page": 4,
        "total": 556,

        "pages": {
            "first": "https://myserver/film-api/1.1/films?year=1975&page=1",
            "prev": "https://myserver/film-api/1.1/films?year=1975&page=3",
            "next": "https://myserver/film-api/1.1/films?year=1975&page=5",
            "last": "https://myserver/film-api/1.1/films?year=1975&page=23",
        }
    },

    "data": [
        {
            "id": 962,
            "name": "Jaws",
            "director": "Steven Spielberg",
            "year": 1975
        },
        ...
    ]
}
~~~

There is some disagreement about what meta information to include and whether it should appear in the body or HTTP headers, but I try not to worry about it too much, and go with what I feel is more intuitive.


# HTTP status codes

A great way to make your API as standardised as possible is to make sure you are returning the correct [HTTP status code](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes), as all clients *should* agree on what these mean.

That being said, you will soon find that some of the codes are far too specific, and some of them are just silly.

The following are the status codes that I tend to use with APIs:

Code | Phrase | Meaning
--- | --- | ---
200 | OK | standard success response
201 | Created | a new resource was created
204 | No Content | nothing to return
400 | Bad Request | invalid request
401 | Unauthorized | no credentials supplied
403 | Forbidden | successful authentication, but user does not have permission for this particular action
404 | Not Found | specified resource does not exist
405 | Method Not Allowed | the HTTP verb is not supported for the resource, or permissable for the user
429 | Too Many Requests | rate-limiting prevented this request
500 | Internal Server Error | server fault
501 | Not Implemented | future feature
503 | Service Unavailable | down for maintenance

## Errors

While HTTP Status codes are great, it is often worth including a more human-readable error message in the body.
This is particularly useful for 4xx errors, where the client is at fault.

Consider making suggestions as to what might have gone wrong, and what the user could do to correct their request or workaround the problem.


# Further reading

- [Vinay Sahni](http://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api) - My favourite go-to reference for RESTful best practices, which covers most of my points in much more detail.


*[HATEOAS]: Hypermedia As The Engine Of Application State
