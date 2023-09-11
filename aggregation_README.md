## using aggregation framework

retrieve data effectively and in a structured way

aggregation is a just alternative to find() method

aggregation pipeline stages

aggregation framework is building a pipeline of steps that run on data and return a result

collections => {$match} => {$sort} => {$group} => {$project} => {$limit} => {$skip} => {$out}

every stage receives the output from previous stage and passes it to the next stage

#### importing persons data
```shell
docker cp persons.json mongodb:/tmp/persons.json
docker exec mongodb mongoimport -d analytics -c persons --drop --file /tmp/persons.json --jsonArray
```

instead of using find() method, we can use aggregation framework
it takes an array of stages
```shell
db.collections.aggregate.help()
db.collection.aggregate(pipeline, options)
#Calculates aggregate values for the data in a collection or a view.
#For more information on usage: https://docs.mongodb.com/manual/reference/method/db.collection.aggregate

```
first step will fetch data from the database and can take advantages of indexes
```js
// you can type command in multiple lines just by not closing the parenthesis or brackets db.persions.aggregate([ then anter and type ...
// every step is an document
// first step here is to match the data, just like filter in find() method
db.persons.aggregate([
 {$match: {gender: "female"}} 
])

//group stage
db.persons.aggregate([
  {$match: {gender: "female"}},
  {$group: { _id: {stage: "$location.state"}, totalPersons: {$sum: 1}}}
])

//sort the result of group stage
db.persons.aggregate([
  {$match: {gender: "female"}},
  {$group: { _id: {stage: "$location.state"}, totalPersons: {$sum: 1}}},
  {$sort: {totalPersons: -1}}
])

// example 
db.persons.aggregate([
    { $match: { 'dob.age': { $gt: 50 } } },
    {
      $group: {
        _id: { gender: '$gender' },
        numPersons: { $sum: 1 },
        avgAge: { $avg: '$dob.age' }
      }
    },
    { $sort: { numPersons: -1 } }
  ]).pretty();
 
 
```

#### $project stage

project works same way as projection in find() method
can add new fields to the result
```js
db.persons.aggregate([
  {$project: {_id: 0, gender: 1, fullName: {$concat: ["$name.first", " ", "$name.last"]}}}
])

// transforming data first and last name to uppercase
db.persons.aggregate([
  {$project: {_id: 0, gender: 1, fullName: {$concat: [{$toUpper: "$name.first"}, " ", {$toUpper: "$name.last"}]}}}
])

// transforming data first and last name to first letter uppercase
db.persons.aggregate([
  {$project: {_id: 0, gender: 1, fullName: {$concat: [
    {$toUpper: {$substrCP: ["$name.first", 0, 1]}}, 
    {$substrCP: ["$name.first", 1, {$subtract: [{$strLenCP: "$name.first"}, 1]}]},
    " ",
    {$toUpper: {$substrCP: ["$name.last", 0, 1]}}, 
    {$substrCP: ["$name.last", 1, {$subtract: [{$strLenCP: "$name.last"}, 1]}]},
    
  ]}}}
])
```

turning the Location into geoJSON object
```js
//adding an extra project stage to the previous pipeline

db.persons.aggregate([

  {$project: {_id: 0, name: 1, email: 1, location: {type: "Point", coordinates: [
    "$location.coordinates.longitude",
    "$location.coordinates.latitude"
  ]}}},
  
  {$project: {_gender: 1, email: 1, location: 1, fullName: {$concat: [
    {$toUpper: {$substrCP: ["$name.first", 0, 1]}}, 
    {$substrCP: ["$name.first", 1, {$subtract: [{$strLenCP: "$name.first"}, 1]}]},
    " ",
    {$toUpper: {$substrCP: ["$name.last", 0, 1]}}, 
    {$substrCP: ["$name.last", 1, {$subtract: [{$strLenCP: "$name.last"}, 1]}]},
  ]}}}
  
])

// transforming the coordinates to double
db.persons.aggregate([

  {$project: {_id: 0, name: 1, email: 1, location: {type: "Point", coordinates: [
    {$convert: {input: "$location.coordinates.longitude", to: "double", onError: 0.0, onNull: 0.0}},
    {$convert: {input: "$location.coordinates.latitude", to: "double", onError: 0.0, onNull: 0.0}}
  ]}}},
  
  {$project: {_gender: 1, email: 1, location: 1, fullName: {$concat: [
    {$toUpper: {$substrCP: ["$name.first", 0, 1]}}, 
    {$substrCP: ["$name.first", 1, {$subtract: [{$strLenCP: "$name.first"}, 1]}]},
    " ",
    {$toUpper: {$substrCP: ["$name.last", 0, 1]}}, 
    {$substrCP: ["$name.last", 1, {$subtract: [{$strLenCP: "$name.last"}, 1]}]},
  ]}}}
])


```

transforming the date of birth

you can have separate project stages but lets modify the previous one

```js

db.persons.aggregate([

  {$project: {_id: 0, name: 1, email: 1, 
      birthdate: {$convert: {input: "$dob.date", to: "date"}},
      age: "$dob.age", 
      location: {type: "Point", coordinates: [
    {$convert: {input: "$location.coordinates.longitude", to: "double", onError: 0.0, onNull: 0.0}},
    {$convert: {input: "$location.coordinates.latitude", to: "double", onError: 0.0, onNull: 0.0}}
  ]}}},
  
  {$project: {_gender: 1, email: 1, birthdate: 1, age: 1, location: 1, fullName: {$concat: [
    {$toUpper: {$substrCP: ["$name.first", 0, 1]}}, 
    {$substrCP: ["$name.first", 1, {$subtract: [{$strLenCP: "$name.first"}, 1]}]},
    " ",
    {$toUpper: {$substrCP: ["$name.last", 0, 1]}}, 
    {$substrCP: ["$name.last", 1, {$subtract: [{$strLenCP: "$name.last"}, 1]}]},
  ]}}}
])

```

isoWeekYear operator
```js
db.persons.aggregate([

  {$project: {_id: 0, name: 1, email: 1, 
      birthdate: {$convert: {input: "$dob.date", to: "date"}},
      age: "$dob.age", 
      location: {type: "Point", coordinates: [
    {$convert: {input: "$location.coordinates.longitude", to: "double", onError: 0.0, onNull: 0.0}},
    {$convert: {input: "$location.coordinates.latitude", to: "double", onError: 0.0, onNull: 0.0}}
  ]}}},
  
  {$project: {_gender: 1, email: 1, birthdate: 1, age: 1, location: 1, fullName: {$concat: [
    {$toUpper: {$substrCP: ["$name.first", 0, 1]}}, 
    {$substrCP: ["$name.first", 1, {$subtract: [{$strLenCP: "$name.first"}, 1]}]},
    " ",
    {$toUpper: {$substrCP: ["$name.last", 0, 1]}}, 
    {$substrCP: ["$name.last", 1, {$subtract: [{$strLenCP: "$name.last"}, 1]}]},
  ]}}},

    {$group: {
      _id: {birthYear: {$isoWeekYear: "$birthdate"}},
      numPersons: {$sum: 1}
    }},
    
    {$sort: {numPersons: -1}}
])

```
