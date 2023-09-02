# MongoDB - The Complete Developer's Guide

* run mongodb locally with docker

```docker run --name mongodb -d -p 27017:27017 mongodb/mongodb-community-server```

references: https://www.mongodb.com/docs/manual/tutorial/install-mongodb-enterprise-with-docker/

* run mongo shell

```docker exec -it mongodb mongo```

* show databases

```show dbs```

* create database (if database doesn't exist will be created on demand)

```use flights```

* create collection (if collection doesn't exist will be created on demand) and insert data, _id will be generated automatically
place document inside insertOne method

```db.flights.insertOne({})```

```db.flights.insertOne({"departureAirport":"MUC","arrivalAirport":"SFO","aircraft":"Airbus A380","distance":12000,"intercontinental":true})```

* fetch or find all documents in collection

```db.flights.find().pretty()```

* json vs bson => JSON and BSON are close cousins, as their nearly identical names imply, but you wouldn’t know it by looking at them side by side. JSON, or JavaScript Object Notation, is the wildly popular standard for data interchange on the web, on which BSON (Binary JSON) is based.


* crud operations

create ```insertOne(data, options)``` ```insertMany(data, options)```
read ```find(filter, options)``` ```findOne(filter, options)```
update ```updaetOne(filter, update, options)``` ```updateMany(filter, update, options)``` ```replaceOne(filter, update, options)```
delete ```deleteOne(filter, options)``` ```deleteMany(filter, options)```