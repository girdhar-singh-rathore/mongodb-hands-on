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

### $group vs $project

group is grouping multiple documents into one document, project is one to one relationship
group we do sum, count, average, build array but in project we can only transform data include and exclude fields

#### pushing elements into newly created array

```sh
# importing data
docker cp array-data.json mongodb:/tmp/array-data.json
docker exec mongodb mongoimport -d friendsData -c friends --drop --file /tmp/array-data.json --jsonArray
```

```js

db.friends.aggregate([
    {$group: {_id: {age: "$age"}, allHobbies: {$push: "$hobbies"}}}
])
//unwind the array, it flattens the array
db.friends.aggregate([
  {$unwind: "$hobbies"},
  {$group: {_id: {age: "$age"}, allHobbies: {$push: "$hobbies"}}}
])

// eliminating duplicates, you can use addToSet instead of push
db.friends.aggregate([
  {$unwind: "$hobbies"},
  {$group: {_id: {age: "$age"}, allHobbies: {$addToSet: "$hobbies"}}}
])

//using projection with array
db.friends.aggregate([
  {$project: {_id: 0, examScore: {$slice: ["$examScores", 2, 1]}}}
])

//length of array
db.friends.aggregate([
  {$project: {_id: 0, numScores: {$size: "$examScores"}}}
])

//using filter
db.friends.aggregate([
  {$project: {_id: 0, examScores: {$filter: {
    input: "$examScores",
    as: "sc",
    cond: {$gt: ["$$sc.score", 60]}
  }}}}
])
// apply multiple operations on array
db.friends.aggregate([
    { $unwind: "$examScores" },
    { $project: { _id: 1, name: 1, age: 1, score: "$examScores.score" } },
    { $sort: { score: -1 } },
    { $group: { _id: "$_id", name: { $first: "$name" }, maxScore: { $max: "$score" } } },
    { $sort: { maxScore: -1 } }
]).pretty();

```

#### understading $bucket and $bucketAuto stage

```js

db.persons
  .aggregate([
    {
      $bucket: {
        groupBy: '$dob.age',
        boundaries: [18, 30, 40, 50, 60, 120],
        output: {
          numPersons: { $sum: 1 },
          averageAge: { $avg: '$dob.age' }
        }
      }
    }
  ]).pretty()

db.persons.aggregate([
    {
      $bucketAuto: {
        groupBy: '$dob.age',
        buckets: 5,
        output: {
          numPersons: { $sum: 1 },
          averageAge: { $avg: '$dob.age' }
        }
      }
    }
  ]).pretty()

// example other pipeline stages
db.persons.aggregate([
    { $match: { gender: "male" } },
    { $project: { _id: 0, gender: 1, name: { $concat: ["$name.first", " ", "$name.last"] }, birthdate: { $toDate: "$dob.date" } } },
    { $sort: { birthdate: 1 } },
    { $skip: 10 },
    { $limit: 10 }
]).pretty()

```


#### How MongoDB Optimizes Your Aggregation Pipelines
MongoDB actually tries its best to optimize your Aggregation Pipelines without interfering with your logic.

Learn more about the default optimizations MongoDB performs in this article: https://docs.mongodb.com/manual/core/aggregation-pipeline-optimization/