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

## Create & Insert Documents deep dive

insertOne
```sh
db.collectionName.insertOne({field: value})
```

insertMany
```sh
db.collectionName.insertMany([{field: value}, {field: value}])
```

insert
```sh
db.collectionName.insert()
```

mongoimport
```sh
mongoimport -d cars -c carsList --drop --jsonArray 
```

```sh
use contactData
db.persons.insertOne({name: "Max", age: 29, hobbies: ["Sports", "Cooking"]})
db.persons.insertMany([{name: "Manu", age: 30, hobbies: ["Eating", "Walking"]}, {name: "Anna", age: 27, hobbies: ["Reading", "Running"]}])
```

insert() can be use to insert single or multiple documents
```sh
db.persons.insert({name: "Maria", age: 29, hobbies: ["Sports", "Cooking"]})
```

insert with option ordered false, if one document fails to insert it will continue to insert other documents
```sh
db.customers.insertMany([{_id: "person1", name: "Maria"}, {_id: "person1", name: "Max"}], {ordered: false})
```

insert with option writeConcern, writeConcern is the number of nodes that have to confirm the write operation
```sh
db.customers.insertMany([{_id: "person1", name: "Maria"}, {_id: "person1", name: "Max"}], {writeConcern: {w: 0}})
db.customers.insertMany([{_id: "person1", name: "Maria"}, {_id: "person1", name: "Max"}], {writeConcern: {w: 1}})
```

#### Automicity
In MongoDB, a write operation is atomic on the level of a single document, even if the operation modifies multiple embedded documents within a single document.

When a single write operation (e.g. db.collection.updateMany()) modifies multiple documents, the modification of each document is atomic, but the operation as a whole is not atomic.


```sh

use companyData
db.companies.insertOne({name: "Max", stock: 100, _id: 1})
db.companies.insertMany([{name: "Manu", stock: 100, _id: 2}, {name: "Anna", stock: 100, _id: 3}])
# Error: E11000 duplicate key error collection: companyData.companies index: _id_ dup key: { _id: 1.0 }
db.companies.insertMany([{name: "Maria", stock: 100, _id: 1}, {name: "Chris", stock: 100, _id: 5}])
#set ordered to false, it will add all documents except the one with duplicate key
db.companies.insertMany([{name: "Maria", stock: 100, _id: 1}, {name: "Chris", stock: 100, _id: 5}], {ordered: false})
#set writeConcert journal to false
db.companies.insertOne({name: "Maria", stock: 100, _id: 6}, {writeConcern: {w: 0, j: false}})
#set writeConcert journal to true
db.companies.insertOne({name: "Maria", stock: 100, _id: 7}, {writeConcern: {w: 0, j: true}})
```

#### Import data
```sh
#if you are running mongo in docker, copy the file to docker container
docker cp tv-shows.json mongodb:/tmp/tv-shows.json
#run mongoimport
#>docker exec <container-name-or-id> mongoimport -d <db-name> -c <c-name> --file /tmp/xxx.json

docker exec mongodb mongoimport -d movieData -c movies --drop --file /tmp/tv-shows.json --jsonArray
```

Useful Resources & Links
Helpful Articles/ Docs:

insertOne(): https://docs.mongodb.com/manual/reference/method/db.collection.insertOne/

insertMany(): https://docs.mongodb.com/manual/reference/method/db.collection.insertMany/

Atomicity: https://docs.mongodb.com/manual/core/write-operations-atomicity/#atomicity

Write Concern: https://docs.mongodb.com/manual/reference/write-concern/

Using mongoimport: https://docs.mongodb.com/manual/reference/program/mongoimport/index.html


## Reading Documents with operators

#### methods, filter, operators
```sh
#find is a method
db.collectionName.find()

#filter {field: value}
db.collectionName.find({field: value})
#operators $gt, $gte, $lt, $lte, $ne, $in, $nin, $or, $and, $not, $nor, $exists, $type, $expr, $jsonSchema, $mod, $regex, $text, $where
db.collectionName.find({field: {$gt: value}}
```

#### query selectors
* comparison
* logical
* element
* evaluation
* array
* comments
* geospatial

#### projection operators
* $
* $elemMatch
* $slice
* $meta etc..

##### findOne() and find() methods
find gives us cursor back and we can use it to iterate over the documents
fineone returns first document matching filter

```sh
db.collectionName.findOne(filter, projection)
db.collectionName.find(filter, projection)
```
```sh
db.movies.find({runtime: 60})
```

##### comparison operators example
```sh
db.movies.find({runtime: {$eq: 60}})
db.movies.find({runtime: {$ne: 60}})
db.movies.find({runtime: {$gt: 60}})
db.movies.find({runtime: {$gte: 60}})
```

##### querying embedded Fields and Arrays
```sh
db.movies.find({"rating.average": {$gt: 7}})
db.movies.find({genres: "Drama"})

#check the equality match in array
db.movies.find({genres: ["Drama"]})
db.movies.find({genres: ["Drama", "Horror"]})
db.movies.find({genres: {$all: ["Drama", "Horror"]}})
db.movies.find({genres: {$in: ["Drama", "Horror"]}})
db.movies.find({genres: {$nin: ["Drama", "Horror"]}})
```

##### logical operators
```sh
db.movies.find({$or: [{"rating.average": {$lt: 5}}, {"rating.average": {$gt: 9.3}}]})
db.movies.find({$nor: [{"rating.average": {$lt: 5}}, {"rating.average": {$gt: 9.3}}]})
db.movies.find({$and: [{"rating.average": {$gt: 9}}, {genres: "Drama"}]})
#same as above, default is and

#below is genres will be replaced with horror, it overrides the previous genres
db.movies.find({genres: "Drama", genres: "Horror"}).count()

db.movies.find({"rating.average": {$gt: 9}, genres: "Drama"})
db.movies.find({"rating.average": {$gt: 9}, genres: "Drama"})

db.movies.find({$and: [{"rating.average": {$gt: 9}}, {genres: "Drama"}, {genres: "Horror"}]})
db.movies.find({$and: [{"rating.average": {$gt: 9}}, {genres: {$in: ["Drama", "Horror"]}}]})
db.movies.find({runtime: {$not: {$eq: 60}}}).count()

```

##### element operators
```sh
db.movies.find({summary: {$exists: true}})
db.movies.find({summary: {$exists: false}})
db.movies.find({runtime: {$exists: true, $gte: 90}})
db.movies.find({runtime: {$exists: true, $ne: null, $gte: 90}})
```

##### type operators

```sh
db.movies.find({runtime: {$type: "int"}})
db.movies.find({runtime: {$type: ["int", "string"]}})
```

##### evaluation operators
```sh
db.movies.find({$expr: {$gt: ["$runtime", "$budget"]}})
db.movies.find({$expr: {$gt: ["$runtime", "$boxOffice.budget"]}})
db.movies.find({$expr: {$gt: ["$runtime", {$add: ["$boxOffice.budget", "$boxOffice.domesticGross"]}]}})
```

regex 
```sh
db.movies.find({summary: {$regex: /musical/}})
db.movies.find({summary: {$regex: /musical/, $options: "i"}})
```

expr
```sh
db.movies.find({$expr: {$gt: ["$runtime", "$budget"]}})
db.movies.find({$expr: {$gt: ["$runtime", "$boxOffice.budget"]}})
db.movies.find({$expr: {$gt: ["$runtime", {$add: ["$boxOffice.budget", "$boxOffice.domesticGross"]}]}})

db.sales.find({$expr: {$gt: [{$cond: {if: {$gte: ["$volume", 190]}, then: {$subtract: ["$volume", 30]}, else: "$volume"}}, "$target"]}}).pretty()
```

Excersise
```sh
docker cp boxoffice.json mongodb:/tmp/boxoffice.json
docker exec mongodb mongoimport -d boxofficeData -c movies  --drop --file /tmp/boxoffice.json --jsonArray
use boxofficeData

db.movies.find({"meta.rating": {$gt: 9.2}, "meta.runtime": {$lt: 100}}).count()
db.movies.find({genre: {$in: ["action","drama"]}}).count()

#all the movies where visitor exceeded expectedVisitors

db.movies.find({$expr: {$gt: ["$visitors", "$expectedVisitors"]}}).count()

```

##### querying arrays 
dot operator to access the fields in array like nested documents
```sh
db.movies.find({genre: "Drama"})
db.movies.find({genre: ["Drama", "Horror"]})
db.movies.find({genre: {$all: ["Drama", "Horror"]}})
```

size operator, it looks for exact size of array not greater than or less than
```sh
db.movies.find({genre: {$size: 2}})
```

all operator, it will not check exact match with order
```sh
#it will return count 1
db.movies.find({genre: ["action", "thriller"]}).count()

#it will return count 3
db.movies.find({genre: {$all: ["action", "thriller"]}}).count()
```

elemMatch operator, it will check for exact match in array of objects
```sh
use userData
db.users.insertMany([{name: "Max", hobbies: [{title: "Sports", frequency: 3}, {title: "Cooking", frequency: 6}]}, {name: "Anna", hobbies: [{title: "Sports", frequency: 2}, {title: "Yoga", frequency: 3}]}])
#it will return count 2, below will check frequency in all objects in array, in this case it is checking of Anna's yoga as well
db.users.find({$and: [{"hobbies.title": "Sports"}, {"hobbies.frequency": {$gte: 3}}]}).count()
#it will return count 1
db.users.find({hobbies: {$elemMatch: {title: "Sports", frequency: {$gte: 3}}}}).count()
```

excersise
```sh
# find all movies with exactly 2 genres
db.movies.find({genre: {$size: 2}}).count()
# find all movies which aired in 2018
db.movies.find({"meta.aired": 2018}).count()
# find all movies with ratings greater than 8 and lower than 10 
db.movies.find({"meta.rating": {$gt: 8, $lt: 10}}).count()
```


#### understanding cursor
request the batch of data from the server
shell gives you 20 records by default

```sh
use movieData
db.movies.find().count()
db.movies.find().next()
db.movies.find().next()

#store the result in variable
const dataCursor = db.movies.find()
dataCursor.next()

#iterate over the cursor
dataCursor.forEach(doc => printjson(doc))
dataCursor.hasNext()
```

##### sorting cursor results
sort function takes document which describe how to sort the data, 1 for ascending and -1 for descending
```sh
db.movies.find().sort({"rating.average": 1})
db.movies.find().sort({"rating.average": -1})
db.movies.find().sort({"rating.average": -1, "runtime": 1})
```

##### skipping and limiting results
skip function takes number of documents to skip
```sh
 db.movies.find().sort({"rating.average": -1, "runtime": 1}).skip(100).count()
 db.movies.find().sort({"rating.average": -1, "runtime": 1}).skip(100).limit(10).count()
```

##### using projection to shape our results
```sh
db.movies.find({}, {name: 1, genres: 1, runtime: 1, rating: 1, _id: 0})
db.movies.find({}, {name: 1, genres: 1, runtime: 1, rating: 1, _id: 0}).pretty()
db.movies.find({}, {name: 1, genres: 1, runtime: 1, rating: 1, _id: 0}).sort({"rating.average": -1}).skip(10).limit(10).pretty()
```

projection in array
```sh
db.movies.find({genres: "Drama"},{"genres.$": 1}).pretty()
db.movies.find({genres: {$all: ["Drama", "Horror"]}},{"genres.$": 1}).pretty()
db.movies.find({genres: "Drama"}, {"genres": {$elemMatch: {$eq: "Horror"}}}).pretty()
```

##### understanding slice 
slice operator is used to limit the number of elements in array
```sh
#it will return first 2 elements in array genres
db.movies.find({"rating.average": {$gt: 9}}, {genres: {$slice: 2}}).pretty()
#skip first 1 element and return next 2 elements
db.movies.find({"rating.average": {$gt: 9}}, {genres: {$slice: [1, 2]}, name: 1}).pretty()
```

Useful Resources & Links
Helpful Articles/ Docs:

More on find(): https://docs.mongodb.com/manual/reference/method/db.collection.find/

More on Cursors: https://docs.mongodb.com/manual/tutorial/iterate-a-cursor/

Query Operator Reference: https://docs.mongodb.com/manual/reference/operator/query/

## update operations

updateOne() will udpate the first document matching the filter
```sh
db.collectionName.updateOne(filter, update, options)

db.users.updateOne({_id: ObjectId("64fa26a521d63770a1ff30d9")}, {$set: {hobbies: [{titile: "Sports", frequency: 5}, {title: "Cooking", frequency: 3}, {title: "Hiking", frequency: 1}]}})

#update many 
db.users.updateMany({"hobbies.title": "Sports"}, {$set: {isSporty: true}})

#update multiple fields
db.users.updateOne({_id: ObjectId("64fa26a521d63770a1ff30d9")}, {$set: {age: 40, phone: 12345678990}})

#increment by 2
db.users.updateOne({name: "Manuel"}, {$inc: {age: 2}})
#decrement by 1
db.users.updateO
#inc and set together
db.users.updateOne({name: "Manuel"}, {$inc: {age: 2}}, {$set: {isSporty: false}})
#inc and set on same field will throw error
```

##### min, max and mul
```sh
# update the age only if current age is higher, below will change the age of Chris from 40 to 35
db.users.updateOne({name: "Chris"}, {$min: {age: 35}})

# below will change the age of Chris from 35 to 90, old value is lower than new value
db.users.updateOne({name: "Chris"}, {$max: {age: 90}})

#get rid of field, set it to null
db.users.updateMany({isSporty: true}, {$set: {phone: null}})
#remove field from all documents
db.users.updateMany({}, {$unset: {phone: ""}})


```

##### rename field
```sh
db.users.updateMany({}, {$rename: {age: "totalAge"}})
```

##### understanding upsert
```sh
# upsert will will update the document if it exists, if it doesn't exist it will insert the document
# you can pass third parameter as options, upsert: true
# user Maria doesn't exist, it will not insert/update the document
db.users.updateOne({name: "Maria"}, {$set: {age: 29}})
# user Maria doesn't exist, it will insert the document
db.users.updateOne({name: "Maria"}, {$set: {age: 29}}, {upsert: true})
```

##### updating matched arrays

```sh
# find query
db.users.find({hobbies: {$elemMatch: {title: "Sports", frequency: {$gte: 3}}}}).count()
# update query, it will update the hobbies array with highFrequency field
db.users.updateMany({hobbies: {$elemMatch: {title: "Sports", frequency: {$gte: 3}}}}, {$set: {"hobbies.$.highFrequency": true}})

#update all array docs
db.users.updateMany({totalAge: {$gt: 30}}, {$inc: {"hobbies.$[].frequency": -1}})

```
Finding and updating specific fields
```sh
db.users.updateMany({"hobbies.frequency": {$gt: 2}}, {$set: {"hobbies.$[el].goodFrequency": true}}, {arrayFilters: [{"el.frequency": {$gt: true}}] })
```

Adding elements to arrays
```sh
db.users.updateOne({name: "Maria"}, {$push: {hobbies: {title: "Good Wine", frequency: 1}}})
db.users.updateOne({name: "Maria"}, {$push: {hobbies: {$each: [{title: "Hiking", frequency: 2}, {title: "Swimming", frequency: 3}], $sort: {frequency: -1}}}})
```

removing elements from arrays
```sh
db.users.updateOne({name: "Maria"}, {$pull: {hobbies: {title: "Hiking"}}})
# remove last element from array, 1 is last and -1 is first
db.users.updateOne({name: "Maria"}, {$pop: {hobbies: 1}})
```
understanding addToSet
```sh
# addToSet will add element to array only if it doesn't exist
db.users.updateOne({name: "Maria"}, {$addToSet: {hobbies: {title: "Hiking", frequency: 2}}})
```

Useful Resources & Links
Helpful Articles/ Docs:

Official Document Updating Docs: https://docs.mongodb.com/manual/tutorial/update-documents/

## understanding delete operations

deleteOne
```sh
db.collectionName.deleteOne(filter, options)
db.users.deleteOne({name: "Maria"})
db.users.deleteMany({age: {$gt: 30}, isSporty: true})
db.users.deleteMany({totalAge: {$exists: false}, isSporty: true})

#delete all documents
db.users.deleteMany({})
db.users.drop()

#delete database
db.dropDatabase()
```

Useful Resources & Links
Helpful Articles/ Docs:

Official Document Deletion Docs: https://docs.mongodb.com/manual/tutorial/remove-documents/

