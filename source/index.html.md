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

let data = { 
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
    method: "POST",
    headers: {
        "Content-Type": "application/json",
    },
    body: JSON.stringify(data),
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

let data = { 
                "name": "aname",
                "age": "a number",
                "city": "a city",
                "phonenum": "a number",
                "email": "an email",
                "password": "a password",
}

//this code registers a researcher
var response = fetch('https://research-stream.herokuapp.com/user/registerParticipant ', {
    method: "POST",
    headers: {
        "Content-Type": "application/json",
    },
    body: JSON.stringify(data),
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
    "email":"sppedster2@gmail.com",
    "password":"password"
}

var response = fetch('http://localhost:5000/user/login ', {
    method: "POST",
    headers: {
        "Content-Type": "application/json",
    },
    body: JSON.stringify(data),
})
.then(function(response) {
    return response.text();
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

## logout endpoint

```javascript

var response = fetch('https://research-stream.herokuapp.com/user/logout ', {
        method: "GET",
    })
  .then(function(response) {
    return response.text();
  })

```

This returns a string stateting the user is logged out successfully, there are no parameters, if a user is not logged in the returned string will state this information instead

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
    method: "POST",
    headers: {
        "Content-Type": "application/json",
    },
    body: JSON.stringify(data),
})
  .then(function(response) {
    return response.text();
  })
```

This enpoint updates user information. An array of all the user information (wether its being updated or not) is sent to this endpoint, and it will change the information of the user that is already logged in.

this endpoint returns a success message if the update was successfull

HTTP request

* POST https://research-stream.herokuapp.com/user/update

researcher | participant
--------------------- | ---------
 name | Name
 institution | age
 department | city
 address | phone number
 country | email
 phone number | password
 email
 password

##participant signs up for a study

```javascript

var response = fetch('https://research-stream.herokuapp.com/user/signup/<the id of a study>/<the date the participant is signin up for> ', {
    method: "POST",
    headers: {
        "Content-Type": "application/json",
    },
    body: JSON.stringify(data),
})
  .then(function(response) {
    return response.text();
  })
```

This is the endpoint used when a perticipant is logged in and wants to sign up for a study
it returns a success message if the participant is signed up successfully

a participant cannot sign up for a trial more than once and cannot sign up for a time slot thats aready been filled

### parameters
* _id of the study
* date of the trial

# Studies

## Create studies

> before accessing this endpoint, access the login enpoint with researcher credentials

```javascript

var fileField = document.querySelector("input[type='file'][multiple]"); //use htmp form with <input type="file" />

let data = {
    "name" : "new study",
    "details": "s detail",
    "lab": "a lab",
    "contactInfo" : "some contact information",
    "location" : "a place",
    "compensation": "prob money",
    "criteria" : "a person",
    "expectation" : "an expectation",
    "emaildateoffset": 1,
    "dates": "[01.01.2018, 01.02.2018]"//etc
}

formData.append('ethics', fileField.files[0]);
formData.append('emailcontent', fileField.files[1]);

var response = fetch('https://research-stream.herokuapp.com/study/create ', {
    method: "POST",
    headers: {
        "Content-Type": "application/json",
    },
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
* number of days in advance the reminder e mail will be sent
* an array of the dates the trials will take place
* the ethics clearance file passed to the fetch request through an html form
* a txt file passed through with the content of the reminder email


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
    "ethicsclearance": "buffer file data for the submitted file",
    "emailcontent": "buffer data for a text file containing the e mail content",
    "emaildateoffset": "the number of days an e mail will be sent in advance to warn a participant",
    "dates": "[{date:date of the trial, participant:_id of the participant, emaildate:date the warning e mail will be sent to the participant}]"
}
```

This endpoint return a list of all studies.

accessible by

* https://research-stream.herokuapp.com/study/get

## get Studies by their ID

>the ethics clearance data of a study is stored as buffer data

```javascript

fetch('https://research-stream.herokuapp.com/study/id/<someID>:id', {
        method: "GET",
    })
  .then(function(response) {
    return response.text();
  })

fetch('https://research-stream.herokuapp.com/study/EthicsClearance/<someID>:id', {
        method: "GET",
    })
  .then(function(response) {
    return response;
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

```javascript

fetch('https://research-stream.herokuapp.com/study/name/<someName>', {
        method: "GET"
    })
  .then(function(response) {
    return response.text();
  })

fetch('https://research-stream.herokuapp.com/study/location/<someLocation>', {
        method: "GET"
    })
  .then(function(response) {
    return response.text();
  })

fetch('https://research-stream.herokuapp.com/study/lab/<some lab>', {
        method: "GET"
    })
  .then(function(response) {
    return response.text();
  })

fetch('https://research-stream.herokuapp.com/study/researcher/<some researcher>', {
        method: "GET"
    })
  .then(function(response) {
    return response.text();
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


# Messageing system instruction
## files
1. dump_from_server.js
2. send_msg.js
3. index.js
4. encryption.js

## how to use it
1. when user open the chat page, set up the user and chatroom
2. add this code in the webpage to trigger socket.io
3. dump_from_server.dump will return the chat history, need to insert it into the web page. 
4. encryption haven't been tested yet



# usage of email.js and job.js
### email
this is the method for sending the email, 
open the file and 
  1. set service to the email provider, 
  2. set the user and from to your own email address, 
  3. set the pass to your authentication passcode
NEED TEST!
-----------
run this to send email.

```javascript

const email = require('email.js')
send = new email()
send.setDest("destination email addr")
send.setTxt("path to the text file")
send.setsubject("subject of the email")
send.sendMail()
```

### job
config the heroku scheduler to run this script daily, it will check the database and send email to user who have send_date = today 

NEED TEST! 
*run this locally first to see if it can grab data from database.*


# TODO

1. ~~~Messageing endpoint documentation~~~
3. calander intergration
