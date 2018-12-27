---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - javascript

toc_footers:
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the API documentation page.

This API is currently being hosted on [Heroku](https://research-stream.herokuapp.com/) and uses MongoDB Atlas to host the databases.

This example API documentation page was created with [Slate](https://github.com/lord/slate).


# Users

## Signup endpoints -- researcher

```javascript

let dataResearcher = { 
                "name": "aname",
                "institution": "an institution",
                "department": "department",
                "address": "an address",
                "country": "a country",
                "phonenum": "a number",
                "email": "an email",
                "password": "a password",
}

//this code registers a researcher
var response = fetch('https://research-stream.herokuapp.com/user/registerResearcher ', {
        method: "POST"
        body: JSON.stringify(dataResearcher),
    })
  .then(function(response) {
    return response.json();
  })

```
>Signup endpoints are different for participants and researcher

This function returns a json of the new user information in the form of the dataResearcher variable and prints this json to the console

HTTP request

* POST https://research-stream.herokuapp.com/user/registerResearcher 

###parameters for researchers
* name
* institution
* department
* address
* country
* phone number
* email
* password

## Signup endpoints -- researcher

```javascript

let dataResearcher = { 
                "name": "aname",
                "age": "a number",
                "city": "a city",
                "phonenum": "a number",
                "email": "an email",
                "password": "a password",
}

//this code registers a researcher
var response = fetch('https://research-stream.herokuapp.com/user/registerParticipant ', {
        method: "POST"
        body: JSON.stringify(dataResearcher),
    })
  .then(function(response) {
    return response.json();
  })

```
>Signup endpoints are different for participants and researcher

This function returns a json of the new user information in the form of the dataResearcher variable and prints this json to the console

HTTP requests

* POST https://research-stream.herokuapp.com/user/registerParticipant

###parameters for participants
* Name
* age
* city
* phone number
* email
* password

## Login endpoint

```javascript

let data = { 
                "email": "an email",
                "password": "a password",
}

//this code registers a researcher
var response = fetch('https://research-stream.herokuapp.com/user/login ', {
        method: "POST"
        body: JSON.stringify(data),
    })
  .then(function(response) {
    return response.json();
  })

```

The User is identified by a unique email
The login endpoint saves a user's unique ID to a session state to that it can be checked before the user accesses other checkpoints.
All the endpoints following this one require the login endpoint to be run first

Login is accessed via the endpoint

* https://research-stream.herokuapp.com/user/login

### login parameters
* email
* password

## User Update Endpoint

>to update a participant different data needs to be used but they are both updates using the same endpoint
```javascript
//data variable should be in the same format as the create user data values depending on the type of user
let data = { 
                "name": "aname",
                "age": "a number",
                "city": "a city",
                "phonenum": "a number",
                "email": "an email",
                "password": "a password",
}

//this code registers a researcher
var response = fetch('https://research-stream.herokuapp.com/user/update ', {
        method: "POST"
        body: JSON.stringify(data),
    })
  .then(function(response) {
    return response.json();
  })
```
This enpoint updates user information. It does this by calling the create user function which validates input and, if an object with the same id already exists, performes an update action instead of a create action

this endpoint returns a json of the updated user information

HTTP request

* POST https://research-stream.herokuapp.com/user/update

researcher | participant
-------------------------
 name | Name
 institution | age
 department | city
 address | phone number
 country | email
 phone number | password
 email
 password

# Studies

## Create studies
> before accessing this endpoint, access the login enpoint with researcher credentials
```javascript
let data = {
    "name" : "new study",
    "details": "s detail",
    "lab": "a lab",
    "contactInfo" : "some contact information",
    "location" : "a place",
    "compensation": "prob money",
    "criteria" : "a person",
    "expectation" : "an expectation",
    "path": "<a path to a file>"
}

var response = fetch('https://research-stream.herokuapp.com/study/create ', {
        method: "POST"
        body: JSON.stringify(data),
    })
  .then(function(response) {
    return response.json();
  })

```
Creates a study and saves it to the studies collection of the database.

The email and the name of the researcher is taken from the login information and the field "ethicsClearance" stores a file for the proof of ethics clearance - the file must be less than 14 mb

This enpdoint is also used to update a study, all fields can be changes except the researcher email and name (unless these are fist changed in the user object)

### HTTP request

* POST https://research-stream.herokuapp.com/study/create

###study paratemers
* name
* details
* lab
* contactInfo
* location
* compensation
* criteria
* expectation


## get all studies
```javascript

var response = fetch('https://research-stream.herokuapp.com/study/get', {
        method: "GET"
    })
  .then(function(response) {
    return response.json();
  })

```

>This code return an array of jsons like this

```json
{
    "name": "name",
    "details": "some details",
    "lab":  "a place",
    "contactInfo":  "info",
    "location":  "a city",
    "compensation":  "some form of compensation",
    "criteria":  "study entrant criteria",
    "expectation":  "expectation",
    "email":  "somebody@somewhere.com" ,
    "researcher": "a persons name",
}
```

This endpoint return a list of all studies.

accessible by

* https://research-stream.herokuapp.com/study/get

## get Studies by their ID

>the ethics clearance data of a study is stored as buffer data
```javascript
var responseStudy = fetch('https://research-stream.herokuapp.com/study/<someID>:id', {
        method: "GET"
    })
  .then(function(response) {
    return response.json();
  })

var responseEthics = fetch('https://research-stream.herokuapp.com/study/<someID>:id/EthicsClearance', {
        method: "GET"
    })
  .then(function(response) {
    return response.json();
  })
```
These 2 endpoints return 
a) a study by its ID
B) the Ethics Clearance file for a study according to the study ID

###HTTP requests
* https://research-stream.herokuapp.com/study/id/<someID>:id
* https://research-stream.herokuapp.com/study/EthicsClearance/<someID>:id

###parameters
* ID of an existing study (in the url)

## get studies by other parameters
>before accessing any of the folloing endpoints login
```JavaScript
var responseEthics = fetch('https://research-stream.herokuapp.com/study/name/<someName>', {
        method: "GET"
    })
  .then(function(response) {
    return response.json();
  })

var responseEthics = fetch('https://research-stream.herokuapp.com/study/location/<someLocation>', {
        method: "GET"
    })
  .then(function(response) {
    return response.json();
  })

  var responseEthics = fetch('https://research-stream.herokuapp.com/study/lab/<some lab>', {
        method: "GET"
    })
  .then(function(response) {
    return response.json();
  })

  var responseEthics = fetch('https://research-stream.herokuapp.com/study/researcher/<some researcher>', {
        method: "GET"
    })
  .then(function(response) {
    return response.json();
  }
```

These endpoints return studies by name, location, lab, and researcher, as well as one endpoint to return the proof of ethics clearance for the study matching an id specified.

These studies are returned in a json containging an array of studies in the same format at the endpoint above.

HTTP request | returns
--------------------- | ---------
* https://research-stream.herokuapp.com/study/name/name:<someName> | returns all studies under a certain name
* https://research-stream.herokuapp.com/study/location/location:<someLocation> | returns all studies in a specific location
* https://research-stream.herokuapp.com/study/lab/lab:<someLab> | returns all studies performaed in a specific lab
* https://research-stream.herokuapp.com/study/researcher/researcher:<somereRearcher> | returns all studies created by a specific researcher

###parameters (in urls of requests)
* a name
* a location
* a lab
* a researcher

# TODO

1. Messageing endpoint documentation
3. calander intergration
4. fix error messages to be more consistent and helpful