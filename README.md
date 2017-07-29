# JUR
JSON Uniform Response

[![GitHub tag](https://img.shields.io/github/tag/khalyomede/jur.svg)]()

## Summary
- [What is JUR](#what-is-jur)
- [Standard](#standard)
- [Advice on how to implement this standard](#advice-on-how-to-implement-this-standard)

## What is JUR
JUR has been created to offers a uniform JSON response guideline. This comes from a needs to dispose a constantly same JSON response when getting a message from a JSON REST API. This is the main goal, but this standard can be used for any form of JSON response too. The objective is to let developper focus on things that value the most for them, and automatize or standardize the way they process a JSON response. Its form should always be the same, and developpers should quickly find the main information. The goal of this standard is not only to give the user the information they need, but also to help developers find how their API responded. They should not play hide and seek to find if an attribute "data" is set or not, nor doing complicated and useless computing to find how much time the API last for example. JUR large inspiration comes from JSend standard as it is one of the simplest yet performant standard. Unfortunately, the cons of this standard was mainly its consistency.

[back to summary](#summary)
## Standard
### Uniform response
No matter the HTTP protocol, the response will always looks like this :
```json
{
  "request": "update",
  "status": "success",  
  "requested": "1501325303723",
  "resolved": "1501325303980",
  "elapsed": "257",
  "message": "the resource have successfully been saved",
  "code": 0,
  "data": [
    {
      "firstName": "John",
      "lastName": "Doe",
      "createdAt": "2017-07-28 16:47:00",
      "updatedAt": "2017-07-28 16:47:00"
    }
  ]
}
```

[back to summary](#summary)
### Possible values for the request attribute
This values are included in REST schema.
- `get`: represent the request for getting a resource or a list of resource
- `post`: represent the request for creating a resource
- `put`: represent the request for updating a resource. If you send a `PATCH` request, the value of the `request` attribute should still be `PUT` as the comunity seems to accept both as the action of updating a resource. This rule is set up in order to keep the most consitency possible.
- `delete`: represent the request for deleting a resource

[back to summary](#summary)
### Possible values for the status attribute
- `successs`: the request succeeded and has the expected effect on the resource
- `fail`: the request failed the different filtering/validation mecanism on the request values
- `error`: the request failed due to a server-side error (database outage, violation of a table constraint, ...)

[back to summary](#summary)
### Note on the requested attribute
This represents, in **milliseconds**, the time when the server-side script begins to process the request. The best option to represent this value is to immediately create the variable responsible to log the current millisecond timstamp before any process (if possible).
### Note on the resolved attribute
This represents the time in **milliseconds** when the API is about to send the response back to the consumer system. This should be the last - 1 statement of your server-side code, right before the code that will send the response to the consumer back. A tips is given to make this the most relevant data possible in the [Advice on how to implement this standard](#advice-on-how-to-implement-this-standard) section to help you implement this.
### Note on the elapsed attribute
This represents the time in **milliseconds** the script last to process the request. It simply is the difference between the `requested` and the `resolved` milliseconds timestamps, as they wrap any process that has helped to get the excpected result or effect to the resource. A tips is given to automatically compute this data in the [Advice on how to implement this standard](#advice-on-how-to-implement-this-standard) section to help you simply implement this.
### Notes on the message attribute
It can be anything that help the **end user** to know what happened of his initial request.
#### Success message
This should be as simple as possible, and inform the user that the resource have been successfuly altered.
#### Fail message
This should be as precise as possible, and inform the user that one of the values of his input request was wrong. The recommendation is to display only the first error if multiple values are wrong, as the user will not read everything if you throw every errors. Web experience should be fast, concise, and clear.
#### Error message
Do not throw the exact database driver error or any other system error ! This practice is a huge security breach and could inform bad guys how your system works/is built. Simply display a general error. Here is a suggestion of general error to give your end-user : "an error occured while trying to <name of the action> your data".

[back to summary](#summary)
### Notes on the code attribute
The code should be your way to debug the system in case of an error. It can be given to your end user as you still are the only one that can decrypt your error codes.
#### In case of success
This code should remain at `0`.
#### In case of a failure
This code should be negative and lower than zero. For example, if the attribute `password` and `password_confirm` are not equal, you can throw `-1`. If the password is empty, you could throw `-2`. Another example could be to detect if your database throws a duplicate constraint error. This error could be identifyied as a database, so an error, but as it impact the input values, this is considered a validation error in this case. Eventually, try to be the most specific possible. This is a time loosing job, but trust me when you need to debug your API after 3 month of production, it comes really useful (with the good documentation along of course).
#### In case of an error
This code should be positive and greater than zero. Same rule, try to be the most specific and identifiy each different scenario. However, as there is a tons of system error message, the correct way to do is to only detect the most frequent errors. For example, in MySQL you can have thousands of errors codes. But if you only focus on the main errors, like a foreign key constraint violation (1062), and a database outage, you drastically reduce the number of codes to create. Think smart for this case and be consistent, always to make the debugging easier.

[back to summary](#summary)
### Notes on the data attribute
This is where your resource should be inserted. Ideally, all the request types (`POST`, `GET`, `PUT`, `DELETE`) should return a non empty data. 
#### In case of a GET
Return the single or the list of resources requested.
#### In case of a POST
Return the resource your system have created.
#### In case of a PUT
Alter the resource, and then GET the resource. It can double the time of your API to responds, but it is useful in the case of a concurrent UPDATE.
#### In case of a DELETE
GET the information of the resource, and then DELETE it. You can then return the information of the deleted resource.

[back to summary](#summary)

## Advice on how to implement this standard
- Use constants to define the different values of attributes : if you create your class, set some constants to help the end developper
- Implement a parser : The rule of this parser should be strictly leaded by the presence of all the 5 attributes and their possible values. JCR has been made by thinking of the most uniform data possible, thus making the construction of a parser a breeze. Further version of this documentation should include a pseudo-code algorithm to help implement this parser
- Real-life example : You can find a PHP example of implementation (a little bit hard if you do not do advanced PHP) but this really gives you good hints on how to implement this standard. You can find here : [https://github.com/khalyomede/php-jur/tree/master/src](https://github.com/khalyomede/php-jur/tree/master/src)
- For the `resolved` attribute, you could make the class or the function that send back the response to automatically compute the millisecond timestamp, thus computing right after the `elapsed` value in milliseconds. This can be a good thing in term of Developer Experience. Further version of this documentation should include a pseudo-code algorithm to help you understand how to simply implement this.

[back to summary](#summary)
## Semantic Version ready
This documentation follows the [semantic versioning guidelines](http://semver.org/) v2.0.0 that ensure every step we engage, by adding new functionalities or providing bug fixes, follows these guide and make your job easier by trusting this documentation as versioningly stable.

[back to summary](#summary)
