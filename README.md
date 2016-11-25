![bassmaster Logo](https://raw.github.com/spumko/bassmaster/master/images/bassmaster.png)

Bassmaster makes it easy to combine requests into a single one. It also supports pipelining, allowing you to take the result of one query in the batch request and use it in a subsequent one.  The batch endpoint only responds to POST requests.

[![Build Status](https://secure.travis-ci.org/hapijs/bassmaster.png)](http://travis-ci.org/hapijs/bassmaster)

[![NPM](https://nodei.co/npm/bassmaster.png?downloads=true&stars=true)](https://nodei.co/npm/bassmaster/)

Lead Maintainer: [Christopher De Cairos](https://github.com/cadecairos)


## Getting Started
Install **bassmaster** by either running `npm install bassmaster` in your sites working directory or add 'bassmaster' to the dependencies section of the 'package.json' file and run `npm install`.

### Available options
At this time the options object supports the following configuration:

- `batchEndpoint` - the path where batch requests will be served from.  Default is '/batch'.
- `description` - route description used for generating documentation. Default is 'Batch endpoint'
- `notes` - route notes used for generating documentation. Default is 'A batch endpoint which makes it easy to combine multiple requests to other endpoints in a single call.'
- `tags` - route tags used for generating documentation. Default is ['bassmaster']
- `auth` - If you need the batch route to have authentication

### Examples


The response body to the batch endpoint is an ordered array of the response to each request.  Therefore, if you make a request to the batch endpoint that looks like

```json
{ "requests": [
    {"method": "get", "path": "/users/1"},
    {"method": "get", "path": "/users/2"}
] }
```

The response will look like the following, where the first item in the response array is the result of the request from the first item in the request array.

```json
[{"userId": "1", "username": "bob"}, {"userId": "2", "username": "billy" }]
```

--

When making a POST request as part of the batch assign the _'payload'_ property with the contents of the payload to send.

Optionally you can assign the query as a third property rather than placing it directly into the path. The query property accepts an object that will be formatted into a querystring.

```json
{ "requests": [
    { "method": "get", "path": "/users/1", "query": { "id": "23", "user": "John" } }
] }
```
--

Pipelines can be used in the payloads to save roundtrips to the client. As an example, assume that the server has a route at '/currentuser' and '/users/{id}/profile/'.

You can make a POST request to the batch endpoint with the following body and it will return an array with the current user and their profile.
Pipelining uses [Hoek.reach](https://www.npmjs.com/package/hoek#reach-obj-chain-options) to retrieve values from request results. Where the `$0.id` refers to the array position of the result you wish to query, in this case the contents of the 'currentuser' response.

```json
{ "requests": [
    {"method": "get", "path": "/currentuser"},
    {"method": "get", "path": "/users/$0.id/profile"}
] }
```

-- 

You can use a similar approach when you expect and array response from a request. For example if you are using paging for the query and want to expand the results returned.

In this request the `#0.id` instructs the pipeline to apply to each element returned from the first response. Correspondingly the second element of the response will be an array containing the number of elements returned from the query.

```json
{ "requests": [
    {"method": "get", "path": "/users", "query": { "search": "John", "page": "1", "size": "25" }},
    {"method": "get", "path": "/users/#0.id/profile"}
] }
```

--

If an error occurs as a result of one the requests to an endpoint it will be included in the response in the same location in the array as the request causing the issue.  The error object will include an error property that you can interrogate.  At this time the response is a 200 even when a request in the batch returns a different code.
