# MongoDB - The Complete Developer's Guide

* run mongodb locally with docker

```docker run --name mongodb -d -p 27017:27017 mongodb/mongodb-community-server```

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
db.flights.insertOne({})
```
```sh 
db.flights.insertOne({"departureAirport":"MUC","arrivalAirport":"SFO","aircraft":"Airbus A380","distance":12000,"intercontinental":true})
```

* fetch or find all documents in collection

```sh 
db.flights.find().pretty()
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

