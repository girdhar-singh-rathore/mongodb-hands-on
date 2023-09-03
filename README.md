# MongoDB - The Complete Developer's Guide

* run mongodb locally with docker

```sh 
docker run --name mongodb -d -p 27017:27017 mongodb/mongodb-community-server
 ```

references:
* https://www.mongodb.com/docs/manual/tutorial/install-mongodb-enterprise-with-docker/
* https://www.mongodb.com/docs/drivers/
* https://www.mongodb.com/docs/manual/tutorial/project-fields-from-query-results/

##### run mongo shell
```sh 
docker exec -it mongodb mongo
```
* show databases

```sh 
show dbs
```

* create database (if database doesn't exist will be created on demand)
```sh 
use flights
```

* create collection (if collection doesn't exist will be created on demand) and insert data, _id will be generated automatically
place document inside insertOne method

```sh 
db.flightData.insertOne({})
```
```sh 
db.flightData.insertOne({"departureAirport":"MUC","arrivalAirport":"SFO","aircraft":"Airbus A380","distance":12000,"intercontinental":true})
```

* fetch or find all documents in collection

```sh 
db.flightData.find().pretty()
```

* json vs bson => JSON and BSON are close cousins, as their nearly identical names imply, but you wouldn’t know it by looking at them side by side. JSON, or JavaScript Object Notation, is the wildly popular standard for data interchange on the web, on which BSON (Binary JSON) is based.


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

* delete one
```sh
db.flightData.deleteOne({distance:12000})
```
*delete many
```sh
db.flightData.deleteMany({})
```

* update one, set marker to delete (first insert document)
```sh
db.flightData.insertOne({"departureAirport":"MUC","arrivalAirport":"SFO","aircraft":"Airbus A380","distance":12000,"intercontinental":true})
db.flightData.updateOne({distance:12000}, {$set: {marker: "delete"}})
```
* update all
```sh
db.flightData.updateMany({}, {$set: {marker: "toDelete"}})
```

* delete all
```sh
db.flightData.deleteMany({})
```

* insert many
```sh
db.flightData.insertMany([
    {"departureAirport":"MUC","arrivalAirport":"SFO","aircraft":"Airbus A380","distance":12000,"intercontinental":true},
    {"departureAirport":"LHR","arrivalAirport":"TXL","aircraft":"Airbus A320","distance":950,"intercontinental":false}
])
```

* find all
```sh   
db.flightData.find().pretty()
```
* find all with filter
```sh
db.flightData.find({distance: 950}).pretty()
db.flightData.find({distance: {$gt: 1000}}).pretty()
db.flightData.find({distance: {$gt: 1000}, intercontinental: true}).pretty()
```

* find one, returns first document matching filter
```sh
db.flightData.findOne({distance: 950})
```

* update vs updateMany, update replaces the whole document, updateMany updates all documents matching filter
```sh
```sh
db.flightData.updateOne({distance: 950}, {$set: {delayed: true}})
```

* replaceOne, replaces the whole document
```sh
db.flightData.replaceOne({distance: 950}, {marker: "delete"})
```

* understanding find and cursor object

* add many passengers to flightData
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

* find all passengers and show only name, _id is always shown, if you don't want to show _id you have to explicitly set it to 0
```sh
db.passengers.find({}, {name: 1})
```
* find all passengers and show only name and exclude _id
```sh
db.passengers.find({}, {name: 1, _id: 0})
```

#### embedded documents (nested documents)
The maximum size an individual document can be in MongoDB is 16MB with a nested depth of 100 levels
* insert document with embedded document
```sh
db.flightData.updateMany({}, {$set: {status: {description: "on-time",lastUpdated: "1 hour ago" }}})
db.flightData.updateMany({}, {$set: {status: {description: "on-time",lastUpdated: "1 hour ago", details: {responsible: "Arya"}}}})
```

* working with arrays
```sh
db.passengers.updateOne({name: "Albert Twostone"}, {$set: {hobbies: ["cricket", "running", "travel"]}})
```
* accessing structured data
```sh
db.passengers.findOne({name: "Albert Twostone"}).hobbies
``` 
* query for data in array
```sh
db.passengers.find({hobbies: "travel"}).pretty()
```
* query data in object
```sh
db.flightData.find({"status.description": "on-time"}).pretty()
```
