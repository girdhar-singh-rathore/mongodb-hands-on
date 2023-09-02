# MongoDB - The Complete Developer's Guide

* run mongodb locally with docker

```sh 
docker run --name mongodb -d -p 27017:27017 mongodb/mongodb-community-server
 ```

references: https://www.mongodb.com/docs/manual/tutorial/install-mongodb-enterprise-with-docker/

* run mongo shell
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