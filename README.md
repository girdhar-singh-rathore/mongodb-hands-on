# MongoDB - The Complete Developer's Guide

* run mongodb locally with docker

```sh 
docker run --name mongodb -d -p 27017:27017 mongodb/mongodb-community-server
 ```

references:

https://www.mongodb.com/docs/manual/tutorial/install-mongodb-enterprise-with-docker/

##### run mongo shell
```sh 
docker exec -it mongodb mongo
```

## Understanding the Basics & CRUD Operations

show databases

```sh 
show dbs
```

create database (if database doesn't exist will be created on demand)
```sh 
use flights
```
create collection (if collection doesn't exist will be created on demand) and insert data, _id will be generated automatically
place document inside insertOne method

```sh 
db.flightData.insertOne({})
```
```sh 
db.flightData.insertOne({"departureAirport":"MUC","arrivalAirport":"SFO","aircraft":"Airbus A380","distance":12000,"intercontinental":true})
```

fetch or find all documents in collection

```sh 
db.flightData.find().pretty()
```

json vs bson => JSON and BSON are close cousins, as their nearly identical names imply, but you wouldn’t know it by looking at them side by side. JSON, or JavaScript Object Notation, is the wildly popular standard for data interchange on the web, on which BSON (Binary JSON) is based.


## crud operations

create 
```sh 
insertOne(data, options)
``` 
```sh 
insertMany(data, options)
```
read 
```sh 
find(filter, options)
``` 
```sh 
findOne(filter, options)
```
update 
```sh 
updaetOne(filter, update, options)
``` 
```sh 
updateMany(filter, update, options)
``` 
```sh 
replaceOne(filter, update, options)
```
delete 
```sh 
deleteOne(filter, options)
``` 
```sh
deleteMany(filter, options)
```

delete one
```sh
db.flightData.deleteOne({distance:12000})
```
delete many
```sh
db.flightData.deleteMany({})
```

update one, set marker to delete (first insert document)
```sh
db.flightData.insertOne({"departureAirport":"MUC","arrivalAirport":"SFO","aircraft":"Airbus A380","distance":12000,"intercontinental":true})
db.flightData.updateOne({distance:12000}, {$set: {marker: "delete"}})
```
update all
```sh
db.flightData.updateMany({}, {$set: {marker: "toDelete"}})
```

delete all
```sh
db.flightData.deleteMany({})
```

insert many
```sh
db.flightData.insertMany([
    {"departureAirport":"MUC","arrivalAirport":"SFO","aircraft":"Airbus A380","distance":12000,"intercontinental":true},
    {"departureAirport":"LHR","arrivalAirport":"TXL","aircraft":"Airbus A320","distance":950,"intercontinental":false}
])
```

find all
```sh   
db.flightData.find().pretty()
```
find all with filter
```sh
db.flightData.find({distance: 950}).pretty()
db.flightData.find({distance: {$gt: 1000}}).pretty()
db.flightData.find({distance: {$gt: 1000}, intercontinental: true}).pretty()
```

find one, returns first document matching filter
```sh
db.flightData.findOne({distance: 950})
```

update vs updateMany, update replaces the whole document, updateMany updates all documents matching filter
```sh
```sh
db.flightData.updateOne({distance: 950}, {$set: {delayed: true}})
```

replaceOne, replaces the whole document
```sh
db.flightData.replaceOne({distance: 950}, {marker: "delete"})
```

understanding find and cursor object

add many passengers to flightData
```sh
db.passengers.insertMany([
  {"name":"Max Schwarzmueller","age":29},
  {"name":"Manu Lorenz","age":30},
  {"name":"Chris Hayton","age":35},
  {"name":"Sandeep Kumar","age":28},
  {"name":"Maria Jones","age":30},
  {"name":"Alexandra Maier","age":27},
  {"name":"Dr. Phil Evans","age":47},
  {"name":"Sandra Brugge","age":33},
  {"name":"Elisabeth Mayr","age":29},
  {"name":"Frank Cube","age":41},
  {"name":"Karandeep Alun","age":48},
  {"name":"Michaela Drayer","age":39},
  {"name":"Bernd Hoftstadt","age":22},
  {"name":"Scott Tolib","age":44},
  {"name":"Freddy Melver","age":41},
  {"name":"Alexis Bohed","age":35},
  {"name":"Melanie Palace","age":27},
  {"name":"Armin Glutch","age":35},
  {"name":"Klaus Arber","age":53},
  {"name":"Albert Twostone","age":68},
  {"name":"Gordon Black","age":38}
])
```

find() gives us curser back and we can use it to iterate over the documents
```sh
db.passengers.find()
```
Type "it" for more
```sh
it
```

to get all documents without cursor
```sh
db.passengers.find().toArray()
```
iterate over documents, forEach will fetch each document and print it, it will not load all documents into memory
```sh
db.passengers.find().forEach((passenger) => {printjson(passenger)})
```

### understanding projection
Allowing you to select only the necessary data rather than selecting the whole set of data from the document.
For Example, If a Document contains 10 fields and only 5 fields are to be shown the same can be achieved using the Projections.

**Filter and operators limit the documents that are returned from the query, but projection limits the fields that are returned from the query.**

find all passengers and show only name, _id is always shown, if you don't want to show _id you have to explicitly set it to 0
```sh
db.passengers.find({}, {name: 1})
```
find all passengers and show only name and exclude _id
```sh
db.passengers.find({}, {name: 1, _id: 0})
```

#### embedded documents (nested documents)
The maximum size an individual document can be in MongoDB is 16MB with a nested depth of 100 levels

insert document with embedded document
```sh
db.flightData.updateMany({}, {$set: {status: {description: "on-time",lastUpdated: "1 hour ago" }}})
db.flightData.updateMany({}, {$set: {status: {description: "on-time",lastUpdated: "1 hour ago", details: {responsible: "Arya"}}}})
```

working with arrays
```sh
db.passengers.updateOne({name: "Albert Twostone"}, {$set: {hobbies: ["cricket", "running", "travel"]}})
```
accessing structured data
```sh
db.passengers.findOne({name: "Albert Twostone"}).hobbies
``` 
query for data in array
```sh
db.passengers.find({hobbies: "travel"}).pretty()
```
query data in object
```sh
db.flightData.find({"status.description": "on-time"}).pretty()
```

references: 
* https://www.mongodb.com/docs/drivers/
* https://www.mongodb.com/docs/manual/tutorial/project-fields-from-query-results/

## Schema & Relations: How to Structure Documents

#### Resetting the Database
```sh
use flights
db.dropDatabase()
```

#### Schema
MongoDB enforces no schema! Documents do no have to use the same schema inside of one collection

#### Data Types

MongoDB has a couple of hard limits - most importantly, a single document in a collection (including all embedded documents it might have) must be <= 16mb. Additionally, you may only have 100 levels of embedded documents.

You can find all limits (in great detail) here: https://docs.mongodb.com/manual/reference/limits/

For the data types, MongoDB supports, you find a detailed overview on this page: https://docs.mongodb.com/manual/reference/bson-types/

example:
```sh
use companyData
db.companies.insertOne({ name: "apple inc", isStartup: true, employees: 33, funding: 12345678901234567890, details: { ceo: "Arya" }, tags: [{ title: "super" }, { title: "perfact" }], foundingData: new Date(), insertedAt: new Timestamp() })
```
check data base stats
```sh
db.stats()
```

#### Example exercise => users, posts[comments] collections
```sh
use blog
db.users.insertMany([{ _id: "user1", name: "Max", age: 29, email: "abc@gmail.com" }, { _id: "user2", name: "Manu", age: 30, email: "def@gmail.com" }])
db.users.find().pretty()

db.posts.insertOne({_id: "post1", title: "My first Post", text: "This is my first post", tags: ["new", "tech"], creator: "user1", comments: [{text: "I like this post", author: "user2"}]})
db.posts.find().pretty()
```

#### Adding collection document validation
```sh
db.posts.drop()

db.createCollection("posts", {
    validator: {
        $jsonSchema: {
            bsonType: "object",
            required: ["title", "text", "creator", "comments"],
            properties: {
                title: {
                    bsonType: "string",
                    description: "must be a string and is required"
                },
                text: {
                    bsonType: "string",
                    description: "must be a string and is required"
                },
                creator: {
                    bsonType: "objectId",
                    description: "must be a objectId and is required"
                },
                comments: {
                    bsonType: "array",
                    description: "must be a array and is required",
                    items: {
                        bsonType: "object",
                        required: ["text", "author"],
                        properties: {
                            text: {
                                bsonType: "string",
                                description: "must be a string and is required"
                            },
                            author: {
                                bsonType: "objectId",
                                description: "must be a objectId and is required"
                            }
                        }
                    }
                }
            }
        }
    }
})

db.posts.insertOne({_id: "post1", title: "My first Post", text: "This is my first post", tags: ["new", "tech"], creator: "user1", comments: [{text: "I like this post", author: "user2"}]})
# Error: Document failed validation
#modify the validation


db.runCommand({
    collMod: "posts",
    validator: {

        db.createCollection("posts", {
            validator: {
                $jsonSchema: {
                    bsonType: "object",
                    required: ["title", "text", "creator", "comments"],
                    properties: {
                        title: {
                            bsonType: "string",
                            description: "must be a string and is required"
                        },
                        text: {
                            bsonType: "string",
                            description: "must be a string and is required"
                        },
                        creator: {
                            bsonType: "string",
                            description: "must be a objectId and is required"
                        },
                        comments: {
                            bsonType: "array",
                            description: "must be a array and is required",
                            items: {
                                bsonType: "object",
                                required: ["text", "author"],
                                properties: {
                                    text: {
                                        bsonType: "string",
                                        description: "must be a string and is required"
                                    },
                                    author: {
                                        bsonType: "string",
                                        description: "must be a objectId and is required"
                                    }
                                }
                            }
                        }
                    }
                }
            }
        })

    },
    validationAction: 'warn'
})


db.posts.insertOne({_id: "post1", title: "My first Post", text: "This is my first post", tags: ["new", "tech"], creator: "user1", comments: [{text: "I like this post", author: "user2"}]})

```

Useful Resources & Links
Helpful Articles/ Docs:

The MongoDB Limits: https://docs.mongodb.com/manual/reference/limits/

The MongoDB Data Types: https://docs.mongodb.com/manual/reference/bson-types/

More on Schema Validation: https://docs.mongodb.com/manual/core/schema-validation/


## Exploring The Shell & The Server

help, you can run help at different levels like db.help(), db.collection.help(), db.collection.find().help()
```sh
use posts
db.help()
db.users.help()
db.users.count.help()
```

Helpful Articles/ Docs:

More Details about Config Files: https://docs.mongodb.com/manual/reference/configuration-options/

More Details about the Shell (mongo) Options: https://www.mongodb.com/docs/manual/reference/method/

More Details about the Server (mongod) Options: https://docs.mongodb.com/manual/reference/program/mongod/

