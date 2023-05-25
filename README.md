# MongoDB

## Introduction
* MongoDB is a type of document DB (NoSQL)
  * Stores data in documents and collections
  * Can be JSON/BSON/YAML
* E.g. instead of customer data in rows can have a individual documents for each customer
  * Documents stored inside collection
  * Fileds not dependent on one another

## Advantages
* Flexibility/scalability
  * Changing a single document don't impact others
  * Easy scaling up for growth
* Popularity
  * Can be used for web/mobile/desktop apps
  * Large community and resources
* Cloud Tooling
  * MongoDB Atlas is multi-cloud DB service
  * MongoDB realm  helps devs build apps integrated with MongoDB

## MongoDB Data
* Collections/Documents
  * We could have a collection each for purchase, inventory and customer data
    * Within customer collection we could have personal info of a specfic customer in a single document
  * MongoDB database is a few collections assembled together to store data for use case
    * E.g. sports store
  * MongoDB instance can support multiple DBs
* Data as JSON
  * MongoDB calls key:value pairs field:value pairs
  * Gives us readability and flexibility
  * Downsides to storing data as JSON
    * Parsing is inefficient from computational perspective
    * Storage wise also not efficient
    * Supports limited data types i.e. dates not supported
* BSON - MongoDBs Storage Format
  * Stands for Binary Javascript Object Notation
  * Differences to JSON
    * Not human-readable
    * More efficient storage wise
    * Supports extra data formats like dates
  * Data stored as BSON internally but users can create/maninpulate data as JSON
    * Best of both worlds
    * E.g.
```
\x00\x00\x00\x02name\x00\a\x00\x00\x00Rodney\x00\x02occupation\x00\r\x00\x00\x00photographer\x00\x10year_of_experience\x00\a\x00\x00\x00\x00
```

## MongoDB Data Modelling Basics
* **Deciding on structure of data is essentially data modeling**
  * Also entails choosing relationships
  * Will impact scalability, maintainability and performance
* **Importance of Data modelling**
  * Model is like a blueprint for data
  * We might have to choose between data integrity vs performance
* **Modeling relationships in MongoDB**
  * Embedded Documents Vs References
* **Embedded Documents**
  * We nest data related to document inside it
    * Called sub-document
  * Good use cases
    * Relationships where one entity contains another i.e. one-to-one
      * E.g. Relationship between car and license plate
        * A car only has one unqiue licence plate
    * Relationships where we have one entity to many sub-entities i.e. one-to-many
      * E.g. One person owns multiple cars
  * In e.g. below car engine data is nested inside car document
    * A.k.a denormalised data model
    * i.e. when related data lumped into single collection
```
// Car Document
{
  car_id: 48273
  model_name: "Corvette",
  engine: {
    engine_power: 490,
    engine_type: "V8",
    acceleration: "High"
  }
}
```
* **References**
  * Allows us to define relationships by creating links between data
  * We split data into multiple documents
  * When we find related data via link it's essentailly normalised data model
    * Mimics how RDB creates data relationships
  * Use cases
    * Many-to-many relationships
      * E.g. Car rentals and individuals renting cars
        * Car can be rented by multiple individuals and individual can rent multiple cars
  * In e.g. below we have engine data in separate collection buy linked into car collection via **engine_id**
```
//Car Document
{
  car_id: 48273
  model_name: "Corvette",
  engine_id: 2165
}
 
// Engine Document
{
  id: 2165
  engine_power: 490,
  engine_type: "V8",
  acceleration: "High"
}
```
* **Choosing The Right Model**
  * **Example Case A:** A time management application that stores one user per task. We want to store details about the task, such as the task name, the task due date, and the user assigned to the task (and their associated details). There can only be one person assigned to each task. - **Embedded Document - one-to-one**
  * **Example Case B:** A contact information management application that can store multiple addresses per user. The application would store important details for the person such as their name, as well as their associated addresses. - **Embedded document unless we have repitition - one-to-many**
  * **Example Case C:** A school registration application that manages multiple students. Each student can be in multiple classes. Each class record can easily identify which students are registered and each student record can quickly find any associated classes. **References - many-to-many**

### CRUD I: FINDING DOCUMENTS
#### Browsing & Selecting Collections
* We can have multiple DBs on single MongoDB instance
  * E.g. one for ecommerce shop, one for blog app etc.
* `show dbs` command will print a list of all DBs in MongoDB instance
```
test> show dbs
admin        40.00 KiB
config       72.00 KiB # Internal use - We usually don't manually put data in here
local        80.00 KiB # Stores data used in replication process and instance specific-data
restaurants  44.00 KiB
```
* To choose a DB to work utilise the `use` command
```
test> use restaurants
switched to db restaurants
restaurants>
```
#### Introduction to Querying
* We can use the `.find()` method on a specific collection
  * Cursor returned
    * An object that points to matched documents
  * We are returned results in **batches**
    * Use `it` keyword to see next batch
      * Stands for iterate
  * E.g.
```
restaurants> db.listingsAndReviews.find()
[
  {
    _id: ObjectId("5eb3d668b31de5d588f43081"),
    address: {
      building: '543',
      coord: [ -73.9922175, 40.7543506 ],
      street: '8 Avenue',
      zipcode: '10018'
    },
    borough: 'Manhattan',
    cuisine: 'American',
    grades: [
      {
        date: ISODate("2014-12-29T00:00:00.000Z"),
        grade: 'A',
        score: 7
      },
...
```
#### Querying Collections
* We can pass queries into the `.find()` method
  * As 1st argument
  * Only documents that match criteria are returned
  * Syntax:
```
db.<collection>.find(
  {
    <field>: <value>,
    <second_field>: <value>
    ...
  }
);
```
  * We can pass in as many fields as we like
    * But they are case sensitive
* E.g.
```
db.auto_makers.find({ country: "Japan" });
```
* find() method uses **operator** under the hood
  * By default implicit $eq operator used

#### Querying Embedded Documents
* Sometimes we may have nested documents a.k.a **sub-documents**
* In the example below models field nests data
```
{
  maker: "Honda",
  country: "Japan",
  models: [
    { name: "Accord" },
    { name: "Civic" },
    { name: "Pilot" },
    …
  ]
},
…
```
* We can query nested data using `.find()` method via **dot-notation** to access nested fields
  * Syntax E.g.
```
db.<collection>.find(
  { 
    "<parent_field>.<embedded_field>": <value> 
  }
)
```
* In the example below we query the nested field of zipcode
```
restaurants> db.listingsAndReviews.find({"address.zipcode": "11231"})
```

#### Comparison Operators: $gt and $lt
* gt stands for **greater than**
  * Conversely lt is **less than**
* Syntax example: `db.<collection>.find( { <field>: { $gt: <value> } } )`
* In E.g below we want to find all parks founded after and not including 1900:
```
{
 name: "Yosemite National Park",
 state: "California",
 founded: 1890
},
{
 name: Crater Lake National Park,
 state: "Oregon",
 founded: 1902
},
{
 name: "Mesa Verde National Park",
 state: "Colorado",
 founded: 1906
},
{
 name: "Olympic National Park",
 state: "Washington",
 founded: 1909
},
…
Query: db.national_parks.find({ founded: { $gt: 1900 }});
```
* Alternative is `$gte`
  * Greater than or equal to
* **We can also use lt or gt on strings**
  * E.g. here returns restaurants with first letter of C or lower i.e. A or B `db.listingsAndReviews.find({"address.street": {$lte: "C"}})`

#### Sorting Documents
* We can use the `.sort()` method to sort our results before we return them
  * Argument is document with fields we want to sort on
    * As well as value
      * Specifying 1 or -1
        * 1 for **ascending**
          * For datetime fields this would be chronological order
        * -1 for **descending**
  * Syntax:
```
db.<collection>.find().sort(
  {
    <field>: <value>,
    <second_field>: <value>,
    …
  }
)
```
Example: `db.records.find().sort({ "release_year": 1 });`
```
{
  _id: ObjectId(...),
  artist: "The Beatles",
  album: "Abbey Road",
  release_year: 1969
},
{
  _id: ObjectId(...),
  artist: "Talking Heads",
  album: "Stop Making Sense",
  release_year: 1984
},
{
  _id: ObjectId(...),
  artist: "Prince",
  album: "Purple Rain",
  release_year: 1984
},
{
  _id: ObjectId(...),
  artist: "Tracy Chapman",
  album: "Tracy Chapman",
  release_year: 1988
}
…
```
* In this example the 2nd and 3rd records may be returned in different ordered as the sorted field is duplicated
  * If we ran the query multiples times
* It's also possible to sort on multiple fields
  * Records will be sorted on first field
    * Then any duplicates will be sorted on 2nd field
  * E.g: `db.records.find().sort({ "release_year": 1,  "artist": 1 });`
    * Documents will be sorted 1st by release year and then for each year value they would be sorted on artist
```
{
  _id: ObjectId(...),
  artist: "The Beatles",
  album: "Abbey Road",
  release_year: 1969
},
{
  _id: ObjectId(...),
  artist: "Prince",
  album: "Purple Rain",
  release_year: 1984
},
{
  _id: ObjectId(...),
  artist: "Talking Heads",
  album: "Stop Making Sense",
  release_year: 1984
},
{
  _id: ObjectId(...),
  artist: "Tracy Chapman",
  album: "Tracy Chapman",
  release_year: 1988
}
…
```

#### Query Projections
* Allows us to return **specfic fields** instead of entire document
  * As we may only need some things
  * Done by passing **second argument** to `.find()` method
  * We give fields a value of 1 or 0
    * 1 means include it
    * 0 means execlude it
* Syntax:
```
db.<collection>.find(
  <query>, 
  { 
    <projection_field_1>: <0 or 1>, 
    <projection_field_2>: <0 or 1>,
    …
  }
)
```
* Example:
```
{
  _id: ObjectId("5eb3d668b31de5d588f4292a"),
  address: {
    building: '2780',
    coord: [ -73.98241999999999, 40.579505 ],
    street: 'Stillwell Avenue',
    zipcode: '11224'
  },
  borough: 'Brooklyn',
  cuisine: 'American',
  grades: [
    { date: ISODate("2014-06-10T00:00:00.000Z"), grade: 'A', score: 5 },
    { date: ISODate("2013-06-05T00:00:00.000Z"), grade: 'A', score: 7 },
    {
      date: ISODate("2012-04-13T00:00:00.000Z"),
      grade: 'A',
      score: 12
    },
    {
      date: ISODate("2011-10-12T00:00:00.000Z"),
      grade: 'A',
      score: 12
    }
  ],
  name: 'Riviera Caterer',
  restaurant_id: '40356018'
}
```
* Query: `db.listingsAndReviews.find( {}, {address: 1, name: 1} )`
* Result: 
```
{
  _id: ObjectId("5eb3d668b31de5d588f4292a"),
  address: {
    building: '2780',
    coord: [ -73.98241999999999, 40.579505 ],
    street: 'Stillwell Avenue',
    zipcode: '11224'
  },
  name: 'Riviera Caterer'
}
```
* The ID field is returned by default but we can turn this off
  * By setting `_id: 0` in projection
    * But can't use inclusion and exclusion at the same time for other fields apart from _id
* We can exclude fields but include all others
  * `db.restaurants.find( {}, {grades: 0} )`


### Other methods
* The `.count()` method returns the number of documents that match a query.
* The `.limit()` method can be chained to the .find() method, we pass in how many documents we want output.
* The `$exists` operator can be included in a query filter to only match documents that contain the given field.
* The `$ne` operator helps check if a field is not equal to a specified value.
* The `$and` and $or operators help perform AND or OR logic operators.
* `.pretty()` method makes query outputs nicer

### CRUD I: Querying on Array Fields
#### Querying for an Entire Array
* We can pass in an array/list to the `.find()` method
  * But the array in the document has to be an **exact match**
    * Arrays with the same elements but different orders and extra elements won't be a match
  * E.g. `db.books.find({ genres: ["young adult", "fantasy", "adventure"] })`
    * Result:
```
{
  _id: ObjectId(...),
  title: "Harry Potter and The Sorcerer's Stone",
  author: "JK Rowling",
  year_published: 1997,
  genres: ["young adult", "fantasy", "adventure"]
},
{
  _id: ObjectId(...),
  title: "The Gilded Ones",
  author: "Namina Forna",
  year_published: 2021,
  genres: ["young adult", "fantasy", "adventure"]
}
```
* If we wanted a match on a particular element in that array we can just pass it into `.find()` instead
  * But it will also match other elements too
* E.g. `db.books.find({ genres: "young adult" })`
  * Result
```
{
  _id: ObjectId(...),
  title: "Children of Blood and Bone",
  author: "Tomi Adeyemi",
  year_published: 2018,
  genres: ["fantasy", "young adult", "adventure"]
},
{
  _id: ObjectId(...),
  title: "The Hunger Games",
  author: "Suzanne Collins",
  year_published: 2008,
  genres: ["young adult", "dystopian", "science fiction"]
},
{
  _id: ObjectId(...),
  title: "The Black Flamingo",
  author: "Dean Atta",
  year_published: 2019,
  genres: ["young adult", "coming of age", "LGBTQ"]
},
…
```
* We can use the `$all` operator to match on multiple elements in array
  * Ignores order or other elements
* E.g. `db.books.find({ genres: { $all: [ "science fiction", "adventure" ] } })`
  * Result:
```
{
  _id: ObjectId(...),
  title: "Jurassic Park",
  author: "Michael Crichton",
  year_published: 1990,
  genres: ["science fiction", "adventure", "fantasy", "thriller"]
},
{
  _id: ObjectId(...),
  title: "A Wrinkle in Time",
  author: "Madeleine L'Engle",
  year_published: 1962,
  genres: ["young adult", "fantasy", "science fiction", "adventure"]
},
{
  _id: ObjectId(...),
  title: "Dune",
  author: "Frank Herbert",
  year_published: 1965,
  genres: ["science fiction", "fantasy", "adventure"]
},
…
```








### Operations
* **Comparison operators on Array fields**
  * `.find()` method can query a collection with comparison operators
  * E.g. below finds document in `ufc_contestants` collections where array field: 2008 > `championship_years` > 2004
```
db.ufc_contestants.find({
  championship_years: { $gt: 2004, $lt: 2008 }
})
```
* **Query projections with .find()**
  * It's possible to limit amount of data returned or return only data that has specific field
    * E.g. below only returns name and location field of documents in restaurants collection with `grade` field
```
db.restaurants.find( { grade: "A" }, { name: 1, location: 1 } )
```
* **Connection to DB**
  * We can navigate to a DB via mongo shell using `mongosh` command
  * In e.g. below we select existing DB called dailychecks
    * `use dailychecks`
  
* **Sort method**
  * `.sort()` method can be appended to queries to order documents
    * 1 used for ascending order
    * -1 for descending order
    * Won't work for fields with duplicate values across documents
    * E.g. below views list of documents in `documentaries` collection sorted in descending order on `release_year` field
```
db.documentaries.find().sort({ release_year: -1 })
```

  

