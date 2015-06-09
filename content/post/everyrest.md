+++
Categories = ["Development", "Golang", "Cloud"]
Description = "An extremely simple RESTful server that anyone can deploy on Google App Engine."
Tags = ["development", "golang", "appengine"]
Title = "EveryREST - Let your users own their data"
Section = "blog"
Date = 2014-08-14T15:40:00Z
+++

# Motivation

A lot of my work is on cloud applications. Developing software to be hosted in the cloud offers many advantages over purchasing and running your own servers. However it also presents a number of challenges. One of the biggest drawbacks relates to the privacy of users' data. A typical approach is to have a multi-tenant data store, such that each user can only access their own data. This architecture has a problem in that the system administrator has access to the whole database and therefore every user's data. The alternative I'm proposing here is that the user should be the only person (assuming we can trust the service provider) with access to their data.

Cloud computing services have become very advanced and easy to use. Why not let each user own their own instance of a database and distribute a front-end separately? With that in mind, I developed [EveryREST](https://github.com/baruchlubinsky/everyrest). 

# Introduction

EveryREST is little more than a RESTful wrapper for the [Google Cloud Datastore](https://developers.google.com/datastore/). By RESTful here I mean designed to be a back-end for [Ember Data](http://emberjs.com/guides/models/), using the [REST Adapter](http://emberjs.com/guides/models/the-rest-adapter/), with minimal configuration. The program does not handle any application logic, that must be done by the client side application (or an intermediate web service). Consequently developers can be confident that the back-end will "never" have to be updated. Any new features are added to the front-end application, which is shared by all users. The only information about users that needs to be stored centrally is reference to their instance of EveryREST, that is then secured by their Google account.

_This is a work in progress. I know that the webservice works, but I have not tried to secure it to using Google acocunts yet._

# API

The web service uses [beerapi](http://godoc.org/github.com/baruchlubinsky/beerapi/api) to handle requests. It responds to the following URLs. Where "beers" is a place holder for any resource name.

### GET /beers/

Returns a list of the beer records. Example response:

{{< highlight json >}}
{
  "beers": [
    {"id": "1", "name": "Castle", "type": "Lager", "comments":["1", "2"]},
    {"id": "2", "name": "Woodhouse", "type": "Ale", "comments":["3", "5", "7"]}
  ]
}
{{< /highlight >}}

### GET /beers/id

Get a single record.

{{< highlight json >}}
{
  "beer": [
    {"id": "1", "name": "Castle", "type": "Lager", "comments":["1", "2"]},
  ]
}
{{< /highlight >}}

### GET /beers?ids[]=1&ids[]=2

Get all the specified records, response is the same as that for GET.

### POST /beers/

Create a new record. Example request payload:

{{< highlight json >}}
{
  "beer": {"name":"Castle","type":"Lager","comments":[]}
}
{{< /highlight >}}

### PUT /beers/id

Update the specified record. Request is the same as POST.

### DELETE /beers/id

Delete the specified record.

# Deploy

Anyone can deploy a version of this web service for themselves.

- Set up the [Go App Engine](https://developers.google.com/appengine/docs/go/gettingstarted/devenvironment) development environment
- Create an account on [App Engine](https://appengine.google.com/)
- Register an application, taking note of the ID
- Clone the [repository](https://github.com/baruchlubinsky/everyrest)
- Replace `everyrest` with your application ID in `app.yaml` and `dispatch.yaml`
- `$ goapp deploy app.yaml`

# Client application

The service doesn't care what kind of application consumes it, and is set up with CORS allowing any IP by default. Checkout my [post](/post/2014/08/beerdemo) for an example of a very simple Ember application powered by this server.
