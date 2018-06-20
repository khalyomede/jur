# JUR

JSON Uniform Response

[![GitHub tag](https://img.shields.io/github/tag/khalyomede/jur.svg)]()

## Summary

- [What is JUR](#what-is-jur)
- [Standard](#standard)
- [Real life example](#real-life-example)
- [Tips](#tips)
- [Attributes cheat sheet](#attributes-cheat-sheet)

You are watching version 2, which supersedes the previous version. Version 1 is no longer maintened (you can still access it through the tag menu right above).

## What is JUR

Json Uniform Response is my attempt at making the job of developers that consumes API easier by a set of defined rules to make the data retrieving task the same regardless of the nature of the data we are trying to fetch or alter. This might seems a little bit abstract said like this, so quickly jump into the [real life example](#real-life-example) if you need some real use cases.

JUR is optimized for REST API that respond with JSON data.

## Standard

- [Overview](#overview)
- [In-depth attributes explaination](#in-depth-attributes-explaination)

### Overview

No matter which endpoint you request, nor which type of HTTP request you make, this is what you will always have in response:

```json
GET http://example.com/api/task

{
  "message": null,
  "request": "get",
  "data": [
    {
      "id": 1,
      "name": "Fix the :hover state of the submit button",
      "updated_at": "2018-06-20T10:07:37+02:00"
    },
    {
      "id": 2,
      "name": "Add Youtube social icon on the menu",
      "updated_at": "2018-06-20T11:29:01+02:00"
    }
  ],
  "debug": {
    "elapsed": 120000,
    "issued_at": 1529843640000000,
    "resolved_at": 1529843640120000
  },
}
```

### In-depth attributes overview

- [message](#message)
- [request](#request)
- [data](#data)
- [debug](#debug)

#### message

This is the right place to inform the end user of what is happening with his request. 

The user can be a developer, but also a non-developer like a customer in a shop website. So this message should be targeted to offer a good comprehension, **without** exposing sensible information about the server or any other critical data.

### request

This is the attribute that saves the type of request that have been processed. This attribute can help developers in 2 ways:

- verify its request have been understood by the server (if the developer intended to make a GET request, but the server ended up processing a PUT request, there is an issue)
- help the front-end developer process the message regarding the type of request (styling, ...)

### data

This is the attribute where all the necessary data to provide are stored. It can be set to null when there is no data to return, which is different than the absence of result (which would be an empty array/object).

### debug

This is the attribute designed for the consumer of the API. It let you know when the request has been issued and resolved according to the server, and how many time did the server took to resolve this request.

**elapsed**

The time elapsed by the controller to resolved the request. In **microseconds**.

**issued_at**

The time at which the request has been handled by the controller. Timestamp in **microseconds**.

**resolved_at**

The time at which the request has been resolved by the controller. Timestamp in **microseconds**.

## Tips

- [Use the debug information to monitor latency](#use-the-debug-information-to-monitor-latency)
- [Version your routes](#version-your-routes)
- [Internationalize your routes](#internationalize-your-routes)

### Use the debug information to monitor latency

One interesting thing about the debug attribute is that you can use it to compute latency between the time your client sent the request until the moment the controller is about to start resolving the request. 

_Keep in mind the request could also go through many layers like framework middlewares, CDNs, ... All of these should be taken into an account._

So for example in Javascript we could compute this like this:

```html
<script type="text/javascript">
  document.addEventListener('DOMContentLoaded', function() {
    const client_issued_at = Date.now() + new Date().getMilliseconds();

    fetch('/api/task', { method: 'POST', body: JSON.stringify({ name: 'rework JUR standard' }) }).then(function(response) {
      const resp = response.json();
      const sever_issued_at = resp.debug.issued_at;
      const latency = client_issued_at - server_issued_at;

      alert(resp.message + 'saved in ' + Math.round(resp.debug.elapsed / 1000) + 'ms (+ ' + Math.round(latency) + 'ms)');
    });
  });
</script>
```

Which would return to the client an alert with the following message:

```
Task "rework JUR standard" successfuly saved in 120ms (+ 90ms)
```

### Version your routes

In case of a breaking change in your tables, you might want to version your API to not force all your front-end developper to immediately change their code. Simply add a `v1`, to your routes.

### Internationalize your routes

As the message attribute is intended for being displayed to the end user, you will also want to personalize the message regarding the locale of the device. For this, I propose you to add the locale in the route like:

- `/api/v1/en-us/task`
- `/api/v1/fr-fr/configuration/notification`
- ...

Specifying the locale in the route instead of on the header or the body/query parameters has the advantage of making this a requirement more than an option, so it is allow the back-end developer to have less verification to do or remove some tasks (like having to split by `;` the language of the `Accept-Language` header, in case there is multiple language, or to interpret `*` and having to fetch the default language, ...).

## Attributes cheat-sheet

| attribute   | parent | type        | format      | example                                                                             |
|-------------|--------|-------------|-------------|-------------------------------------------------------------------------------------|
| message     |        | string|null |             | "Task created successfuly."                                                         |
|             |        |             |             | null                                                                                |
| request     |        | string      |             | get                                                                                 |
|             |        |             |             | post                                                                                |
|             |        |             |             | put                                                                                 |
|             |        |             |             | patch                                                                               |
|             |        |             |             | delete                                                                              |
| data        |        | *           |             | [{"id": 1, "name": "Do the dishes"}, {"id": 2, "name": "Buy catnips"}]              |
|             |        |             |             | null                                                                                |
|             |        |             |             | {"lat": 48.8773406, "lng": "2.327774"}                                              |
| debug       |        | object      |             | {"elapsed": 120000, "isued_at": 1529843640000000, "resolved_at": 1529843640120000"} |
| elapsed     | debug  | integer     | microsecond | 120000                                                                              |
| issued_at   | debug  | integer     | microsecond | 1529843640000000                                                                    |
| resolved_at | debug  | integr      | microsecond | 1529843640120000                                                                    |