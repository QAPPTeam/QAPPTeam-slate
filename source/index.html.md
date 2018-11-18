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

# Index.js and db.js

## API setup code

```javascript
const express = require('express')
var app = express();
const bodyParser = require('body-parser');
var urlencodedParser = bodyParser.urlencoded({ extended: false })
const session = require('express-session');
const connect = require('./db').connect;

app.use(bodyParser.json());
app.use(urlencodedParser);

//app.use(express.static('demo'));

app.use(session({
  secret: 'lets go',
  cookie: {path : '/'},
  resave: true,
  saveUninitialized: false,
  //TODO save sessions to db
}));

//routes for studies researcher and participant
const Study = require('./routes/study.route');
const User = require('./routes/user.route');
app.use('/user', User);
app.use('/study', Study);

app.get('/', function (req, res) {
  res.send('landing page');
});

connect(function(err){
  if(err){
    console.log("error connecting to db")
  } else {
    console.log("connected to db")

  }
});

const PORT = process.env.PORT || 5000
app.listen(PORT, () => console.log(`Listening on ${ PORT }`));
```
This is the main code that sets up the db

The API requires bodyParser in order to properly be able to get data for a front end webpage in a JSON format.

Express-session is used to create a session state for the app to be able to verify that a user is logged in, it is currently being saved to cookies, this will be changed so the ids of logged in users are saved to the database.

The connect function is written in a different document so other endpoints also have access to the open database connection

The app is initiated at the end of this function.

## Connecting to Mongodb Atlas
```javascript
var MongoClient = require('mongodb').MongoClient

let _db

module.exports = {
    get,
    connect
};

function connect(callback) {
  if (_db) return
  MongoClient.connect("mongodb+srv://15lc50:W!ldcats@cluster0-mfedp.mongodb.net/test?retryWrites=true", function(err, client) {
    if (err){
        return callback(err);
    } 
    _db = client;
    return callback(null, _db);
  })
}

function get(){
    if(_db){
        return _db;
    } else {
        console.log("db not set");
    }
}
```
The connect function uses a url aquired from the mongodb Atlas page to connect to the database
in order to connect the ip address of the computer running this code has to be whitelisted in mongoDB Atlas

The get function returns the database connection object for use in other functions

# Users

## User Routes
```javascript
const express = require('express');
const router = express.Router();

// Require the controllers
const userController = require('../controllers/user.controller');

//post request to add users to database
router.post('/registerParticipant', userController.participantCreate);
router.post('/registerResearcher', userController.researcherCreate);
router.post('/login', userController.userLogin);
module.exports = router;
```
User routes, connect user routes to apporpriate user controller

## Researcher Model

> This endpoint is used to verify the inputs when createing a researcher object to save to the database

```javascript
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

let ResearcherSchema = new Schema({
    name: {type: String, required: [true, 'must have a name'], maxlenght: 30},
    institution: {type: String, required: true, maxlenght: 30}, //check if the institution exists?
    department: {type: String, required: true, maxlenght: 30},
    address: {type: String, required: true, maxlenght: 30},
    country: {type: String, required: true, maxlenght: 30},
    phonenum: {type: String, required: true, maxlenght:10},
    email: {type: String, required: true, maxlenght: 30}, //TODO check is e mail and is unique
    password: {type: String, required: true, maxlenght: 100}, //add password confirmation?
    usertype: {type: String, required:true, match:/researcher/}
}); //add better error messages

module.exports = mongoose.model('Researcher', ResearcherSchema);
```

> There are separate user models for researchers and participants

The use of mongoose shemas allows for thorough verification of inputs before they are saved to the database, all the inputs to the schema are currently required

For more information about mongoose schemas [go here](https://mongoosejs.com/docs/schematypes.html).

## Participant Model

> This endpoint is used to verify the inputs when createing a researcher object to save to the database

```javascript
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

let ParticipantSchema = new Schema({
    name: {type: String, required: true, maxlenght: 30},
    age: {type: Number, required: true, max:100},
    city: {type: String, required: true, maxlenght: 30},
    phonenum: {type: String, required: true, maxlenght:10},
    email: {type: String, required: true, maxlenght: 30},  //TODO check e-mail is unique
    password: {type: String, required: true, maxlenght: 100},
    usertype: {type: String, required: true, match:/participant/},
    studies: {type: Array}
});

module.exports = mongoose.model('Researcher', ResearcherSchema);
```

> There are separate user models for researchers and participants

The User model requires fewer inputs than the Researcher model (no department or institution)

'Studies' array will be filled with study objects with specific times the user has signed up for, the calander endpoint is not implemented yet.

The use of mongoose shemas allows for thorough verification of inputs before they are saved to the database, all the inputs to the schema are currently required

For more information about mongoose schemas [go here](https://mongoosejs.com/docs/schematypes.html).

## Signup enpoints

> require the participant and researcher models from above and bcrypt to encrypt passwords before they are saved to the database

```javascript
const Participant = require('../models/participant.model');
const Researcher = require('../models/researcher.model');
const bcrypt = require('bcrypt');

//register user endpoint
exports.participantCreate = function (req, res) {
    var pass = req.body.password
    bcrypt.hash(pass, 10, function (err, hash){
        if (err) {
          return (err);
        }
    let participant = new Participant(
        {
            name: req.body.name,
            age: req.body.age,
            city: req.body.city,
            phonenum: req.body.phonenum,
            email: req.body.email, 
            password: hash,
        },
        //user input not sanatised?
    )
        participant.save(function(err){
            if(err){
                console.log(err)
            } else{
                res.send("participant created successfully")
            }
        })
});
};
//register researcher endpoint
exports.researcherCreate = function (req, res) {
    var pass = req.body.password
    bcrypt.hash(pass, 10, function (err, hash){
        if (err) {
          return (err);
        }
        let researcher = new Researcher(
            {
                name: req.body.name,
                institution: req.body.institution,
                department: req.body.department,
                address: req.body.address,
                phonenum: req.body.phonenum,
                email: req.body.email, //make emails and phone numbers unique
                password: hash,
                usertype: req.body.usertype
            },//dont know what the contraints are for the parameters, will check later
            //user input not sanatised
        )
        researcher.save(function(err){
            if(err){
                console.log(err)
                res.send("error creating researcher");
            } else{
                res.send("researcher created successfully");
            }
        })
    });
}
```

>Signup endpoints are different for participants and researcher (could be combined into one)

The sighup endpoint creates a user ojebct useing the schmeas listed above, hashes the password the user typed in, and saves the user data to the database.

The two signup endpoints are acces via the routes 

* https://research-stream.herokuapp.com/user/registerResearcher 
* https://research-stream.herokuapp.com/user/registerParticipant

## Login endpoint

> requires db to access database and bcrypt the compare hashed and unhashed passwords

```javascript
const db = require("../db");
const bcrypt = require('bcrypt');

exports.userLogin = function (req, res){
    var email = req.body.email;
    var password = req.body.password;
    var connection = db.get();
    connection.db("ResearchDB").collection("Users").findOne({ email: email }, function (err, user) {
        if (err) {
            return (err)
        } else if(req.session.userId){
            console.log("user is already logged in");
        } else if (!user) {
            var err = new Error('User not found.');
            err.status = 401;
            return (err);
        } else {
        bcrypt.compare(password, user.password, function (err, result) {
            if (result === true) {
                res.send("passwords match");
            } else {
                res.send("passwords dont match");
            };
            });
            req.session.userId = user._id
            //TODO save user information to session
        }
          });
}
```

The User is identified by a unique email and password
The login endpoint saves a user's unique ID to a session state to that it can be checked before the user accesses other checkpoints.

Login is accessed via the endpoint

* https://research-stream.herokuapp.com/user/login

# Studies

## Study Routes

```javascript
const express = require('express');
const router = express.Router();
var session = require('express-session');

function requiresLogin(req, res, next) {
    if (req.session && req.session.userId) {
      return next();
    } else {
        console.log(req.session.userId);
        var err = new Error('You must be logged in to view this page.');
        err.status = 401;
        return next(err);
    }
  }

// Require the controllers
const studyController = require('../controllers/study.controller');

// post endpoint to add study to database
router.post('/create', requiresLogin, studyController.studyCreate);
router.get('/get', requiresLogin, studyController.studyGet)
module.exports = router;
```
Connect Study routes to apporpriate study controller

the function requiresLogin runs before the study endpoints are called to ensure that a user is logged in

## Study model

```javascript
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

let StudySchema = new Schema({
    name: {type: String, required: [true, 'must enter a name'], maxlength: 30},
    details: {type: Number, required: true, maxlength:300},
    lab: {type: Number, required: true, maxlength:100},
    contactInfo: {type: String, required: true, maxlength: 30},
    location: {type: String, required: true, maxlength:100},
    compensation: {type: String, required: true, maxlength: 100},
    criteria: {type:String, required: true, maxlength: 100},
    expectation: {type:String, required: true, maxlength: 100},
    email: {type:String, required: true, maxlength: 100},
    researcher: {type:String, required: true}
});

module.exports = mongoose.model('Study', StudySchema);
```
Model for studies, based on provided schema

For more information about mongoose schemas [go here](https://mongoosejs.com/docs/schematypes.html).

## Create studies
```javascript
const Study = require('../models/study.model');

exports.studyCreate = function (req, res) {
    let study = new Study(
        {
            name: req.body.name,
            details: req.body.details,
            lab:  req.body.lab,
            contactInfo:  req.body.contactInfo,
            location:  req.body.location,
            compensation:  req.body.compensation,
            criteria:  req.body.criteria,
            expectation:  req.body.expectation,
            email:  req.body.email,
            researcher: req.body.researcher, //TODO change this to automatically get researcher's name from login information
        },//add error if user is not a researcher
        //user input not sanatised
    )
        study.save(function(err){
            if(err){
                console.log(err)
                res.send("error Saveing study to DB");
            } else{
                res.send("study created successfully");
            }
        })
};
```
Creates a study according to the [study model](#study-model) and saves it to the studies collection of the database.

This endpoint can be accessed through

* https://research-stream.herokuapp.com/study/get

## get all studies
```javascript
const db = require('../db');

exports.studyGet = function (req, res) {
    db.get().db("ResearchDB").collection('Studies').find({}).toArray(function(err, result) {
        if (err) throw err;
        res.send(result);
    });
    //returns all studies, queries needed for returning only studies with certain attributes
};
```

>This code return a json like this

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
It need to be modefied to return an amount and an endpoint needs to be added to get studies matching certain criteria

accessible by

* https://research-stream.herokuapp.com/study/get

# TODO

1. Messageing endpoint documentation
2. Endpoint to get only some studies
3. calander intergration
4. fix error messages to be more consistent and helpful