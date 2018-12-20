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
mongoose

let _db

module.exports = {
    get,
    connect
};

function connect(callback) {
  if (_db) return
  mongoose.connect("mongodb+srv://15lc50:W!ldcats@cluster0-mfedp.mongodb.net/test?retryWrites=true", function(err, client) {
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
in order to connect the ip address of the computer running this code has to be whitelisted in mongoDB Atlas. It connects useing the mongoose connect function so that mongoose schemas can be used when saveing users and studies.

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
router.post('/update', userController.userUpdate);
module.exports = router;
```
User routes, connect user routes to apporpriate user controller.

The User Routes incluse 2 routes to register participants and researchers, one endpoint for logging users in and one for updateing user information.

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

module.exports = mongoose.model('Researcher', ResearcherSchema, "User");
```

> There are separate user models for researchers and participants

The use of mongoose shemas allows for thorough verification of inputs before they are saved to the database.

Validation parameters | effect
--------------------- | ---------
type | The type of item, all items in this schema are strings
required | This input is required, all items in this schema are required
maxlength | The max length of a string
match | item must max the regex statement

For more information about mongoose schemas [go here](https://mongoosejs.com/docs/schematypes.html).

Items created useing this model are saved to the User collection in the database

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

module.exports = mongoose.model('Researcher', ResearcherSchema, "User");
```

> There are separate user models for researchers and participants

The User model requires fewer inputs than the Researcher model (no department or institution)

'Studies' array will be filled with study objects with specific times the user has signed up for, the calander endpoint is not implemented yet.

The use of mongoose shemas allows for thorough verification of inputs before they are saved to the database.

Validation parameters | effect
--------------------- | ---------
type | The type of item, the three kinds of items in this schema are strings, numbers, and arrays
required | This input is required, all items in this schema are required
maxlength | The max length of a string
match | item must max the regex statement

participant model is always saved to the User collection

For more information about mongoose schemas [go here](https://mongoosejs.com/docs/schematypes.html).

## Signup enpoints

> require the participant and researcher models from above and bcrypt to encrypt passwords before they are saved to the database

```javascript
const Participant = require('../models/participant.model');
const Researcher = require('../models/researcher.model');
const bcrypt = require('bcrypt');

//register user endpoint
//register user endpoint
function participantCreate (req, res) {
    var pass = req.body.password
    bcrypt.hash(pass, 10, function (err, hash){
        if (err) {
          console.log(err);
        }
    let participant = new Participant(
        {
            name: req.body.name,
            age: req.body.age,
            city: req.body.city,
            phonenum: req.body.phonenum,
            email: req.body.email,  //make e mails and phone numbers unique
            password: hash,
        },//dont know what the contraints are for the parameters, will add later
        //user input not sanatised?
    )

    var newParticipant = {};
    newParticipant = Object.assign(newParticipant, participant._doc);
    delete newParticipant._id;
  
    Participant.findOneAndUpdate({_id: ObjectId(req.session.userId)}, newParticipant, {upsert: true}, function (err) {
         if (err) console.log(err)
         else res.send("success")
    });
});
};

//register researcher endpoint
function researcherCreate(req, res) {
    var pass = req.body.password
    bcrypt.hash(pass, 10, function (err, hash){
        if (err) {
          console.log(err,500);
        }
        let researcher = new Researcher(
            {
                name: req.body.name,
                institution: req.body.institution,
                department: req.body.department,
                address: req.body.address,
                country: req.body.country,
                phonenum: req.body.phonenum,
                email: req.body.email, //make emails and phone numbers unique
                password: hash,
                usertype: req.body.usertype
            },//dont know what the contraints are for the parameters, will check later
            //user input not sanatised
        )
        var newResearcher = {};
        newResearcher = Object.assign(newResearcher, researcher._doc);
        delete newResearcher._id;
      
        Researcher.findOneAndUpdate({_id: ObjectId(req.session.userId)}, newResearcher, {upsert: true}, function (err) {
             if (err) console.log(err)
             else res.send("success")
        });
    });
}
```

>Signup endpoints are different for participants and researcher

The sighup endpoint creates a user ojebct useing the schmeas listed above, hashes the password the user typed in, and saves the user data to the database.

The two signup endpoints are acces via the routes 

* https://research-stream.herokuapp.com/user/registerResearcher 
* https://research-stream.herokuapp.com/user/registerParticipant

## Login endpoint

> requires db to access database and bcrypt the compare hashed and unhashed passwords

```javascript
function userLogin(req, res){
    var email = req.body.email;
    var password = req.body.password;
    db.get().collection("User").findOne({ email: email }, function (err, user) {
        if (err) return error
        else if(req.session.userId){
            console.log("User is already logged in");
        } else if (!user) {
            return ("Username or password is incorrect",406);
        } else {
        bcrypt.compare(password, user.password, function (err, result) {
            if (result === true) {
                res.send("Logged in");
            } else {
                res.send("Username or password is incorrect",406);
            };
            });
            req.session.userId = user._id
        }
        });

}
```

The User is identified by a unique email and password
The login endpoint saves a user's unique ID to a session state to that it can be checked before the user accesses other checkpoints.

Login is accessed via the endpoint

* https://research-stream.herokuapp.com/user/login

## User Update Endpoint

```javascript
function userUpdate(req,res){
    user = db.get().collection("User").findOne({_id : req.session.userId}, function(err){
    if(err) return (err,500)
    else if(isEmpy(user)){
        return new Error("This user no longer exists",406);
    }
    });
    if(user.usertype = "researcher"){
        researcherCreate(req, res);
    } else{
        participantCreate(req,res);
    }

}
```

This enpoint updates user information. It does this by calling the create user endpoint which validates input and, if an object with the same id already exists, performes and update action instead of a create action (mongoose takes care of this useing the user.save())

# Studies

## Study Routes

```javascript
const express = require('express');
const router = express.Router();

function requiresLogin(req, res, next) {
    if (req.session && req.session.userId) {
      return next();
    } else {
        return ('You must be logged in to view this page.', 401);
    }
  }

// Require the controllers
const studyController = require('../controllers/study.controller');

// post endpoint to add study to database
router.post('/create', requiresLogin, studyController.studyCreate);
router.get('/get', requiresLogin, studyController.studyGet);
router.get('/:id/EthicsClearance', requiresLogin, studyController.getEthics);
router.get('/id/:id', requiresLogin, studyController.getId);
router.get('/name/:name', requiresLogin, studyController.getName);
router.get('/location/:location', requiresLogin, studyController.getLocation);
router.get('/lab/:lab', requiresLogin, studyController.getLab);
router.get('/researcher/:researcher', requiresLogin, studyController.getResearcher);
module.exports = router;
```
Connect Study routes to apporpriate study controller

the function requiresLogin runs before the study endpoints are called to ensure that a user is logged in

## Study model

```javascript
let StudySchema = new Schema({
    name: {type: String, required: [true, 'Must enter a name'], maxlength: 30},
    details: {type: Number, required: true, maxlength:300}, //TODO add better  error messages
    lab: {type: Number, required: true, maxlength:100},
    contactInfo: {type: String, required: true, maxlength: 30},
    location: {type: String, required: true, maxlength:100},
    compensation: {type: String, required: true, maxlength: 100},
    criteria: {type:String, required: true, maxlength: 100},
    expectation: {type:String, required: true, maxlength: 100},
    email: {type:String, required: true, maxlength: 100},
    researcher: {type:String, required: true},
    ethicsClearace: {contentType:String, data:Buffer},//for uploading the ethics clearance file
});

module.exports = mongoose.model('Study', StudySchema, "Study");
```
Model for studies, based on provided schema

Validation parameters | effect
--------------------- | ---------
type | The type of item, the three kinds of items in this schema are strings and numbers
required | This input is required, all items in this schema are required
maxlength | The max length of a string
match | item must max the regex statement

For more information about mongoose schemas [go here](https://mongoosejs.com/docs/schematypes.html).

## Create studies
```javascript
const Study = require('../models/study.model');

exports.studyCreate = function (req, res) {
    reser = db.get().collection("User").findOne({_id : req.session.userId}, function(err){
        if (err) return(err,500);
        else if (reser.usertyepe != "researcher"){
            return new error("Only researcher can create studies", 405)
        }
    });
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
            email:  reser.email,
            researcher: reser.name,
        })
        study.ethicsClearance.data = fs.readFileSync(req.body.path);
        study.ethicsClearance.contentType = req.body.path.split('.').pop();;

        
            //user input not sanatised?
            var newStudy = {};
            newStudy = Object.assign(newStudy, study._doc);
            delete newStudy._id;
          
            Study.findOneAndUpdate({_id: ObjectId(req.body._id)}, newStudy, {upsert: true}, function (err) {
                 if (err) console.log(err)
                 else res.send("success")
            });
    })
};
```
Creates a study according to the [study model](#study-model) and saves it to the studies collection of the database.

The email and the name of the researcher is taken from the login information and the field "ethicsClearance" stores a file for the proof of ethics clearance

This enpoint is also used to update a study, all fields can be changes except the researcher email and name (unless these are fist changed in the user object)

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
The endpoints for returning only some studies depending on certain attributes

accessible by

* https://research-stream.herokuapp.com/study/get

## get some studies

```javascript

exports.getEthics = function (req,res) {
    db.get().collection("Study").findOne({_id :ObjectId(req.params.id)}, function(err,study){
        if (err) console.log(err,500);
        else{
            res.contentType(study.ethicsClearance.contentType);
            res.json(study.ethicsClearance.data);
        }
    })
};

exports.getId = function (req,res) {
    db.get().collection("Study").findOne({_id:ObjectId(req.params.id)}, function(err,study){
        if (err) console.log(err,500);
        else{
            res.send(study);
        }
    })
};

exports.studyGet = function (req, res) {
    db.get().collection("Study").find({}).toArray(function(err, result) {
        if (err) console.log(err,500);
        res.send(result);
    });
    //returns all studies, queries needed for returning only studies with certain attributes
};

//name
exports.getName = function (req, res) {
    db.get().collection("Study").find({name:req.params.name}).toArray(function(err, result) {
        if (err) console.log(err,500);
        else{
            res.send(result.length + " studies have this name\n" + result);
        }
    });
    //lists the number of studies with the specified name and returns studies with a specific name
};

//location
exports.getLocation = function (req, res) {
    db.get().collection("Study").collection('Studies').find({location:req.params.location}).toArray(function(err, result) {
        if (err) console.log(err,500);
        else{
            res.send(result.length + " studies in this location\n" + result);
        }
    });
    //lists the number of studies with the specified locaation and returns list of studies with this name
};

//lab
exports.getLab = function (req, res) {
    db.get().collection("Study").db("ResearchDB").collection('Studies').find({lab:req.params.lab}).toArray(function(err, result) {
        if (err) console.log(err,500);
        else{
            res.send(result.length + " studies executed in this lab\n" + result);
        }
    });
    //lists the number of studies with the specified locaation and returns list of studies with this name
};

//researcher
exports.getResearcher = function (req, res) {
    db.get().collection("Study").db("ResearchDB").collection('Studies').find({researcher:req.params.researcher}).toArray(function(err, result) {
        if (err) console.log(err,500);
        else{
            res.send(result.length + " studies conducted by " + researcher + "\n" + result);
        }
    });
    //lists the number of studies with the specified locaation and returns list of studies with this name
};
```

These endpoints return studies by id, name, location, lab, and researcher, as well as one endpoint to return the proof of ethics clearance for the study matching an id specified.

These studies are returned in a json containging an array of studies in the same format at the endpoint above.

endpoint | returns
--------------------- | ---------
* https://research-stream.herokuapp.com/study/id:<someID>/EthicsClearance | returns the proof of ethics clearance for a study with a specific id
* https://research-stream.herokuapp.com/study/id:<someID> | returns a study by a specific ID (should only be one)
* https://research-stream.herokuapp.com/study/name:<someName> | returns all studies under a certain name
* https://research-stream.herokuapp.com/study/location:<someLocation> | returns all studies in a specific location
* https://research-stream.herokuapp.com/study/lab:<someLab> | returns all studies performaed in a specific lab
* https://research-stream.herokuapp.com/study/researcher:<somereRearcher> | returns all studies created by a specific researcher

# TODO

1. Messageing endpoint documentation
3. calander intergration
4. fix error messages to be more consistent and helpful