**Table of Contents**  *generated with [DocToc](http://doctoc.herokuapp.com/)*

	- [Summary](#summary)
		- [Quick Reference](#quick-reference)
			- [Installation](#installation)
			- [Configuration](#configuration)
	- [Objects](#objects)
		- [Creating Objects](#creating-objects)
		- [Retrieving Objects](#retrieving-objects)
		- [Updating Objects](#updating-objects)
			- [Counters](#counters)
			- [Arrays](#arrays)
			- [Relations](#relations)
- [TODO: This method is not yet implemented.](#todo:-this-method-is-not-yet-implemented)
		- [Deleting Objects](#deleting-objects)
- [TODO: This method is not yet implemented.](#todo:-this-method-is-not-yet-implemented)
		- [Batch Operations](#batch-operations)
- [making a few GameScore objects](#making-a-few-gamescore-objects)
		- [Data Types](#data-types)
			- [Dates](#dates)
			- [Bytes](#bytes)
			- [Pointers](#pointers)
			- [Relation](#relation)
- [TODO: There is no Ruby object representation of this type, yet.](#todo:-there-is-no-ruby-object-representation-of-this-type-yet)
			- [Future data types and namespacing](#future-data-types-and-namespacing)
	- [Queries](#queries)
		- [Basic Queries](#basic-queries)
		- [Query Contraints](#query-contraints)
		- [Queries on Array Values](#queries-on-array-values)
		- [Relational Queries](#relational-queries)
		- [Counting Objects](#counting-objects)
		- [Compound Queries](#compound-queries)
	- [Users](#users)
		- [Signing Up](#signing-up)
		- [Logging In](#logging-in)
	- [Roles](#roles)
	- [Files](#files)
	- [Push Notifications](#push-notifications)
	- [Installations](#installations)
	- [Geopoints](#geopoints)

## Summary

parse-ruby-client lets you interact with Parse using Ruby. There are many uses. For example:

- A webserver can show data from Parse on a website.
- You can upload large amounts of data that will later be consumed in a mobile app.
- You can download recent data to run your own custom analytics.
- You can export all of your data if you no longer want to use Parse.

**Table of Contents**  *generated with [DocToc](http://doctoc.herokuapp.com/)*

  - [Summary](#summary)
    - [Quick Reference](#quick-reference)
      - [Installation](#installation)
      - [Configuration](#configuration)
  - [Objects](#objects)
    - [Creating Objects](#creating-objects)
    - [Retrieving Objects](#retrieving-objects)
    - [Updating Objects](#updating-objects)
      - [Counters](#counters)
      - [Arrays](#arrays)
      - [Relations](#relations)
- [TODO: This method is not yet implemented.](#todo:-this-method-is-not-yet-implemented)
    - [Deleting Objects](#deleting-objects)
- [TODO: This method is not yet implemented.](#todo:-this-method-is-not-yet-implemented)
    - [Batch Operations](#batch-operations)
- [making a few GameScore objects](#making-a-few-gamescore-objects)
    - [Data Types](#data-types)
      - [Dates](#dates)
      - [Bytes](#bytes)
      - [Pointers](#pointers)
      - [Relation](#relation)
- [TODO: There is no Ruby object representation of this type, yet.](#todo:-there-is-no-ruby-object-representation-of-this-type-yet)
      - [Future data types and namespacing](#future-data-types-and-namespacing)
  - [Queries](#queries)
    - [Basic Queries](#basic-queries)
    - [Query Contraints](#query-contraints)
  - [Users](#users)
  - [Roles](#roles)
  - [Files](#files)
  - [Push Notifications](#push-notifications)
  - [Installations](#installations)
  - [Geopoints](#geopoints)

### Quick Reference

#### Installation

`gem install parse-ruby-client` or add `gem "parse-ruby-client"` to your `Gemfile.`

#### Configuration

```ruby
require 'parse-ruby-client'

Parse.init :application_id => "<your_app_id>",
           :api_key        => "<your_api_key>"
```

## Objects

The design philosophy behind parse-ruby-client is to stay out of the way as much as possible. Parse Objects, at the most basic level, act like Ruby `Hash` objects with Parse-specific methods tacked on.

### Creating Objects

```ruby
game_score = Parse::Object.new("GameScore")
game_score["score"] = 1337
game_score["playerName"] = "Sean Plott"
game_score["cheatMode"] = false
result = game_score.save
puts result
```

This will return:

```ruby
{"score"=>1337,
 "playerName"=>"Sean Plott",
 "cheatMode"=>false,
 "createdAt"=>"2013-01-19T21:01:33.562Z",
 "objectId"=>"GeqPWJdNqp"}
```

### Retrieving Objects

The easiest way to retrieve Objects is with `Parse::Query`:

```ruby
game_score_query = Parse::Query.new("GameScore")
game_score_query.eq("objectId", "GeqPWJdNqp")
game_score = game_score_query.get
puts game_score
```

This will return:

```ruby
[{"score"=>1337,
  "playerName"=>"Sean Plott",
  "createdAt"=>"2013-01-19T21:01:33.562Z",
  "updatedAt"=>"2013-01-19T21:01:33.562Z",
  "objectId"=>"GeqPWJdNqp"}]
```

Notice that this is an `Array` of results. For more information on queries, see TODO.

When retrieving objects that have pointers to children, you can fetch child objects by setting the `include` attribute. For instance, to fetch the object pointed to by the "game" key:

```ruby
game_score_query = Parse::Query.new("GameScore")
game_score_query.eq("objectId", "GeqPWJdNqp")
game_score_query.include = "game"
game_score = game_score_query.get
```

You can include multiple children pointers by separating key names by commas:

```ruby
game_score_query.include = "game,genre"
```

### Updating Objects

To change the data on an object that already exists, just call `Parse::Object#save` on it. Any keys you don't specify will remain unchanged, so you can update just a subset of the object's data. For example, if we wanted to change the score field of our object:

```ruby
game_score = Parse::Query.new("GameScore").eq("objectId", "GeqPWJdNqp").get.first
game_score["score"] = 73453
result = game_score.save
puts result
```

This will return:

```ruby
{"score"=>73453,
 "playerName"=>"Sean Plott",
 "createdAt"=>"2013-01-19T21:01:33.562Z",
 "updatedAt"=>"2013-01-19T21:16:34.395Z",
 "objectId"=>"GeqPWJdNqp"}
```

#### Counters

To help with storing counter-type data, Parse provides the ability to atomically increment (or decrement) any number field. So, we can increment the score field like so:

```ruby
game_score = Parse::Query.new("GameScore").eq("objectId", "GeqPWJdNqp").get.first
game_score["score"] = Parse::Increment.new(1)
game_score.save
```

You can also use `Parse::Decrement.new(amount)`.

#### Arrays

To help with storing array data, there are three operations that can be used to atomically change an array field:

1. `Parse::Object#array_add(field, value)` appends the given array of objects to the end of an array field.
2. `Parse::Object#array_add_unique(field, value)` adds only the given objects which aren't already contained in an array field to that field. The position of the insert is not guaranteed.
3. `Parse::Object#array_remove(field, value)` removes all instances of each given object from an array field.

Each method takes an array of objects to add or remove in the "objects" key. For example, we can add items to the set-like "skills" field like so:

```ruby
game_score = Parse::Query.new("GameScore").eq("objectId", "5iEEIxM4MW").get.first
game_score.array_add_unique("skills", ["flying", "kungfu"])
game_score.save
puts game_score["skills"]
```

This will return:

```ruby
[["flying", "kungfu"]]
```

#### Relations

In order to update Relation types, Parse provides special operators to atomically add and remove objects to a relation. So, we can add an object to a relation like so:

```ruby
game_score = Parse::Query.new("GameScore").eq("objectId", "5iEEIxM4MW").get.first
player = Parse::Query.new("Player").eq("objectId", "GLtvtEaGKa").get.first
game_score.array_add_relation("opponents", player.pointer)
game_score.save
game_score["opponents"] #=> #<Parse::ArrayOp:0x007fbe98931508 @operation="AddRelation", @objects=[Player:GLtvtEaGKa]>
game_score["opponents"].objects.first #=> Player:GLtvtEaGKa
```

To remove an object from a relation, you can do:

```ruby
# TODO: This method is not yet implemented.
```

### Deleting Objects

To delete an object from the Parse Cloud, call `Parse::Object#parse_delete`. For example:

```ruby
game_score = Parse::Query.new("GameScore").eq("objectId", "5iEEIxM4MW").get.first
game_score.parse_delete
Parse::Query.new("GameScore").eq("objectId", "5iEEIxM4MW").get.length #=> 0
```

You can delete a single field from an object by using the `Parse::Object#delete_field` operation:

```ruby
# TODO: This method is not yet implemented.
```

### Batch Operations

To reduce the amount of time spent on network round trips, you can create, update, or delete several objects in one call, using the batch endpoint.

parse-ruby-client provides a "manual" way to construct Batch Operations, as well as some convenience methods. The commands are run in the order they are given. For example, to create a couple of GameScore objects using the "manual" style, use `Parse::Batch#add_request`. `#add_request` takes a `Hash` with `"method"`, `"path"`, and `"body"` keys that specify the HTTP command that would normally be used for that command. 

```ruby
batch = Parse::Batch.new
batch.add_request({
  "method" => "POST",
  "path"   => "/1/classes/GameScore"
  "body"   => {
    "score"      => 1337,
    "playerName" => "Sean Plott"
  }
})
batch.add_request({
  "method" => "POST",
  "path"   => "/1/classes/GameScore"
  "body"   => {
    "score"      => 1338,
    "playerName" => "ZeroCool"
  }
})
batch.run!
```

Because manually constructing `"path"` values is repetitive, you can use `Parse::Batch#create_object`, `Parse::Batch#update_object`, and `Parse::Batch#delete_object`. Each of these methods takes an instance of `Parse::Object` as the only argument. Then you just call `Parse::Batch#run!`. For example:

```ruby
# making a few GameScore objects
game_scores = [1, 2, 3, 4, 5].map do |i|
  gs = Parse::Object.new("GameScore")
  gs["score"] = "#{i}"
  gs
end

batch = Parse::Batch.new
game_scores.each do |gs|
  batch.create_object(gs)
end
batch.run!
```

The response from batch will be an `Array` with the same number of elements as the input list. Each item in the `Array` with be a `Hash` with either the `"success"` or `"error"` field set. The value of success will be the normal response to the equivalent REST command:

```ruby
{
  "success" => {
    "createdAt" => "2012-06-15T16:59:11.276Z",
    "objectId"  => "YAfSAWwXbL"
  }
}
```

The value of `"error"` will be a `Hash` with a numeric code and error string:

```ruby
{
  "error" => {
    "code"  => 101,
    "error" => "object not found for delete"
  }
}
```

### Data Types

So far we have only used values that can be encoded with standard JSON. The Parse mobile client libraries also support dates, binary data, and relational data. In parse-ruby-client, these values are encoded as JSON hashes with the __type field set to indicate their type, so you can read or write these fields if you use the correct encoding. See https://github.com/adelevie/parse-ruby-client/blob/master/lib/parse/datatypes.rb for implementation details of several common data types.

#### Dates

Use `Parse::Date::new` to create a date object:

```ruby
date_time = DateTime.now
parse_date = Parse::Date.new(date_time)
```

Dates are useful in combination with the built-in createdAt and updatedAt fields. For example, to retrieve objects created since a particular time, just encode a Date in a comparison query:

```ruby
game_score = Parse::Query.new("GameScore").tap do |q|
  g.greater_than("createdAt", Parse::Object.new(DateTime.now)) # query options explained in more detail later in this document
end.get.first
```

`Parse::Date::new` can take a `DateTime`, iso `Hash`, or a `String` that can be parsed by `DateTime#parse` as the sole argument.

The `Parse::Date` API is not set in stone and will likely change following the suggestions discussed here: https://github.com/adelevie/parse-ruby-client/issues/35. The current methods probably will not go away, but some newer, easier methods will be added.

#### Bytes

`Parse::Bytes` contains an attribute, `base64`, which contains a base64 encoding of binary data. The specific base64 encoding is the one used by MIME, and does not contain whitespace.

```ruby
data = "TG9va3MgbGlrZSB5b3UgZm91bmQgYW4gZWFzdGVyIEVnZy4gTWF5YmUgaXQn\ncyB0aW1lIHlvdSB0b29rIGEgTWluZWNyYWZ0IGJyZWFrPw==\n" # base64 encoded data
bytes = Parse::Bytes.new(data)
```

#### Pointers

The `Pointer` type is used when mobile code sets a `PFObject` (iOS SDK) or `ParseObject` (Android SDK) as the value of another object. It contains the `className` and `objectId` of the referred-to value.

```ruby
pointer = Parse::Pointer.new({"className => gameScore", "objectId" => "GeqPWJdNqp"})
```

Pointers to `user` objects have a `className` of `_User`. Prefixing with an underscore is forbidden for developer-defined classes and signifies the class is a special built-in.

If you already have a `Parse::Object`, you can get its `Pointer` very easily:

```ruby
game_score.pointer
```

#### Relation

The `Relation` type is used for many-to-many relations when the mobile uses `PFRelation` (iOS SDK) or `ParseRelation` (Android SDK) as a value. It has a `className` that is the class name of the target objects.

```ruby
# TODO: There is no Ruby object representation of this type, yet.
```

#### Future data types and namespacing

Though this is something parse-ruby-client will take care for you, it's worth noting:

When more data types are added, they will also be represented as hashes with a `__type` field set, so you may not use this field yourself on JSON objects.


## Queries

Queries are created like so:

```ruby
query = Parse::Query.new("GameScore")
```



### Basic Queries

You can retrieve multiple objects at once by calling `#get`:

```ruby
query.get
```

The return value is an `Array` of `Parse::Object` instances:

```ruby
[{"score"=>100,
  "player"=>player:qPHDUbBbjj,
  "createdAt"=>"2012-10-10T00:16:10.846Z",
  "updatedAt"=>"2012-10-10T00:16:10.846Z",
  "objectId"=>"6ff54A5OCY"},
 {"score"=>1337,
  "playerName"=>"Sean Plott",
  "createdAt"=>"2013-01-05T22:51:56.033Z",
  "updatedAt"=>"2013-01-05T22:51:56.033Z",
  "objectId"=>"MpPBAHsqNg"},
 {"score"=>1337,
  "playerName"=>"Sean Plott",
  "createdAt"=>"2013-01-05T22:53:22.609Z",
  "updatedAt"=>"2013-01-05T22:53:22.609Z",
  "objectId"=>"T1dj8cWwYJ"}]]
```

### Query Contraints

There are several ways to put constraints on the objects found, using various methods of `Parse::Query`. The most basic is `Parse::Query#eq`:

```ruby
query = Parse::Query.new("GameScore").eq("playerName", "Sean Plott")
```

Other constraint methods include:

<table>
  <tr>
    <td>`Parse::Query#less_than(field, value)`</td>
    <td>Less Than</td>
  </tr>
  <tr>
    <td>`Parse::Query#less_eq(field, value)`</td>
    <td>Less Than or Equal To</td>
  </tr>
  <tr>
    <td>`Parse::Query#greater_than(field, value)`</td>
    <td>Greater Than</td>
  </tr>
  <tr>
    <td>`Parse::Query#greater_eq(field, value)`</td>
    <td>Greater Than Or Equal To</td>
  </tr>
  <tr>
    <td>`Parse::Query#not_eq(field, value)`</td>
    <td>Not Equal To</td>
  </tr>
  <tr>
    <td>`Parse::Query#value_in(field, values)`</td>
    <td>Contained In</td>
  </tr>
  <tr>
    <td>`Parse::Query#value_not_in(field, values)`</td>
    <td>Not Contained in</td>
  </tr>
  <tr>
    <td>`Parse::Query#exists(field, value=true)`</td>
    <td>A value is set for the key</td>
  </tr>
  <tr>
    <td>`Parse::Query#select`</td>
    <td>TODO: `$select` not yet implemented. This matches a value for a key in the result of a different query</td>
  </tr> 
</table>

For example, to retrieve scores between 1000 and 3000, including the endpoints, we could issue:

```ruby
scores = Parse::Query.new("GameScore").tap do |q|
  q.greater_eq("score", 1000)
  q.less_eq("score", 3000)
end.get
```

To retrieve scores equal to an odd number below 10, we could issue:

```ruby
scores = Parse::Query.new("GameScore").tap do |q|
  q.value_in("score", [1,3,5,7,9])
end.get
```

To retrieve scores not by a given list of players we could issue:

```ruby
scores = Parse::Query.new("GameScore").tap do |q|
  q.value_not_in("playerName", ["Jonathan Walsh","Dario Wunsch","Shawn Simon"])
end.get
```

To retrieve documents with the score set, we could issue:

```ruby
scores = Parse::Query.new("GameScore").tap do |q|
  q.exists("score") # defaults to `true`
end.get
```

To retrieve documents without the score set, we could issue:

```ruby
scores = Parse::Query.new("GameScore").tap do |q|
  q.exists("score", false)
end.get
```

If you have a class containing sports teams and you store a user's hometown in the user class, you can issue one query to find the list of users whose hometown teams have winning records. The query would look like:

```ruby
users = Parse::Query.new("_User").tap do |users_query|
  users_query.eq("hometown", {
    "$select" => {
      "query" => {
        "className" => "Team",
        "where" => {
          "winPct" => {"$gt" => 0.5}
        }
      },
    "key" => "city"
    }
  })
end.get
```

Currently, there is no convenience method provided for `$select` queries. However, they are still possible. This is a good example of the flexibility of parse-ruby-client. You usually do not need to wait for a feature to be added in order to user it. If you have a good idea on what a convencience method for this should look like, please file an issue, or even better, submit a pull request.

You can use the `Parse::Query#order_by` method to specify a field to sort by. By default, everything is ordered ascending. Thus, to retrieve scores in ascending order:

```ruby
scores = Parse::Query.new("GameScore").tap do |q|
  q.order_by = "score"
end.get
```

And to retrieve scores in descending order:

```ruby
scores = Parse::Query.new("GameScore").tap do |q|
  q.order_by = "score"
  q.order = :descending
end.get
```

You can sort by multiple fields by passing order a comma-separated list. Currently, there is no convenience method to accomplish this. However, you can still manually construct an `order` string. To retrieve documents that are ordered by scores in ascending order and the names in descending order:

```ruby
scores = Parse::Query.new("GameScore").tap do |q|
  q.order_by = "score,-name"
end.get
```

You can use the `limit` and `skip` parameters for pagination. `limit` defaults to 100, but anything from 1 to 1000 is a valid limit. Thus, to retrieve 200 objects after skipping the first 400:

```ruby
scores = Parse::Query.new("GameScore").tap do |q|
  q.limit = 200
  q.skip = 400
end.get
```

All of these parameters can be used in combination with each other.

### Queries on Array Values

For keys with an array type, you can find objects where the key's array value contains 2 by:

```ruby
randos = Parse::Query.new("RandomObject").eq("arrayKey", 2).get
```

### Relational Queries

There are several ways to issue queries for relational data. For example, if each `Comment` has a `Post` object in its `post` field, you can fetch comments for a particular `Post`:

```ruby
comments = Parse::Query.new("Comment").tap do |q|
  q.eq("post", Parse::Pointer.new({
    "className" => "Post",
    "objectId"  => "8TOXdXf3tz"
  }))
end.get
```

If you want to retrieve objects where a field contains an object that matches another query, you can use the `Parse::Query#in_query(field, query=nil)` method. Note that the default limit of 100 and maximum limit of 1000 apply to the inner query as well, so with large data sets you may need to construct queries carefully to get the desired behavior. For example, imagine you have `Post` class and a `Comment` class, where each `Comment` has a relation to its parent `Post`. You can find comments on posts with images by doing:

```ruby
comments = Parse::Query.new("Comment").tap do |comments_query|
  comments_query.in_query("post", Parse::Query.new("Post").tap do |posts_query|
    posts_query.exists("image")
  end)
end.get
```

Note: You must pass an instance of `Parse::Query` as the second argument for `Parse::Query#query_in`. You cannot manually construct queries for this.

TODO: Implement this:
```
If you want to retrieve objects where a field contains an object that does not match another query, you can use the $notInQuery operator. Imagine you have Post class and a Comment class, where each Comment has a relation to its parent Post. You can find comments on posts without images by doing:
```

If you want to retrieve objects that are members of `Relation` field of a parent object, you can use the `Parse::Query#related_to(field, value)` method. Imagine you have a `Post `class and `User` class, where each `Post` can be liked by many users. If the `Users` that liked a Post was stored in a `Relation` on the post under the key likes, you, can the find the users that liked a particular post by:

```ruby
users = Parse::Query.new("_User").tap do |q|
  q.related_to("likes", Parse::Pointer.new({
    "className" => "Post",
    "objectId" => "8TOXdXf3tz"
  }))
end.get
```

In some situations, you want to return multiple types of related objects in one query. You can do this by passing the field to include in the `include` parameter. For example, let's say you are retrieving the last ten comments, and you want to retrieve their related posts at the same time:

```ruby
comments = Parse::Query.new("Comment").tap do |q|
  q.order_by = "createdAt"
  q.order    = :descending
  q.limit    = 10
  q.include  = "post"
end.get
```

Instead of being represented as a `Pointer`, the `post` field is now expanded into the whole object. `__type` is set to `Object` and `className` is provided as well. For example, a `Pointer` to a `Post` could be represented as:

```ruby
{
  "__type" => "Pointer",
  "className" => "Post",
  "objectId" => "8TOXdXf3tz"
}
```

When the query is issued with an `include` parameter for the key holding this pointer, the pointer will be expanded to:

```ruby
{
  "__type" => "Object",
  "className" => "Post",
  "objectId" => "8TOXdXf3tz",
  "createdAt" => "2011-12-06T20:59:34.428Z",
  "updatedAt" => "2011-12-06T20:59:34.428Z",
  "otherFields" => "willAlsoBeIncluded"
}
```

You can also do multi level includes using dot notation. If you wanted to include the post for a comment and the post's author as well you can do:

```ruby
comments = Parse::Query.new("Comment").tap do |q|
  q.order_by = "createdAt"
  q.order    = :descending
  q.limit    = 10
  q.include  = "post.author"
end.get
```

You can issue a query with multiple fields included by passing a comma-separated list of keys as the include parameter:

```ruby
comments = Parse::Query.new("Comment").tap do |q|
  q.include("post,author")
end.get
```

### Counting Objects

If you are limiting your query, or if there are a very large number of results, and you want to know how many total results there are without returning them all, you can use the `count` parameter. For example, if you only care about the number of games played by a particular player:

```ruby
count = Parse::Query.new("GameScore").tap do |q|
  q.eq("playerName", "Jonathan Walsh")
  q.limit = 0
  q.count
end.get
```

With a nonzero limit, that request would return results as well as the count.

### Compound Queries

If you want to find objects that match one of several queries, you can use `Parse::Quer#or` method, with an `Array` as its value. For instance, if you want to find players with either have a lot of wins or a few wins, you can do:

```ruby

players = Parse::Query.new("Player").tap do |q|
  q.greater_than("wins", 150)
  q.or(Parse::Query.new("Player").tap do |or_query|
    or_query.less_than("wins, 5")
  end)
end.get
```

## Users

Many apps have a unified login that works across the mobile app and other systems. Accessing user accounts through parse-ruby-client lets you build this functionality on top of Parse.

In general, users have the same features as other objects, such as the flexible schema. The differences are that user objects must have a username and password, the password is automatically encrypted and stored securely, and Parse enforces the uniqueness of the `username` and `email` fields.

### Signing Up

Signing up a new user differs from creating a generic object in that the `username` and `password` fields are required. The password field is handled differently than the others; it is encrypted when stored in the Parse Cloud and never returned to any client request.

You can ask Parse to verify user email addresses in your application settings page. With this setting enabled, all new user registrations with an `email` field will generate an email confirmation at that address. You can check whether the user has verified their `email` with the `emailVerified` field.

To sign up a new user, create a new `Parse::User` object and then call `#save` on it:

```ruby
user = Parse::User.new({
  :username => "cooldude6", 
  :password => "p_n7!-e8",
  :phone => "415-392-0202"
})
user.save
```

The response body is a `Parse::User` object containing the `objectId`, the `createdAt` timestamp of the newly-created object, and the `sessionToken` which can be used to authenticate subsequent requests as this user:

```ruby
{"username"=>"cooldude6",
 "phone"=>"415-392-0202",
 "createdAt"=>"2013-01-31T15:22:40.339Z",
 "objectId"=>"2bMfWZQ9Ob",
 "sessionToken"=>"zrGuvs3psdndaqswhf0smupsodflkqbFdwRs"}
```

### Logging In

After you allow users to sign up, you need to let them log in to their account with a username and password in the future. To do this, call `Parse::User#authenticate(username, password)`:

```ruby
user = Parse::User.authenticate("cooldude6", "p_n7!-e8")
```

The response body is a `Parse::User` object containing all the user-provided fields except `password`. It also contains the `createdAt`, `updatedAt`, `objectId`, and `sessionToken` fields:

```ruby
{"username"=>"cooldude6",
 "phone"=>"415-392-0202",
 "createdAt"=>"2013-01-31T15:22:40.339Z",
 "updatedAt"=>"2013-01-31T15:22:40.339Z",
 "objectId"=>"2bMfWZQ9Ob",
 "sessionToken"=>"uvs3aspasdnlksdasqu178qaq0smupso"}
```

## Roles

## Files

## Push Notifications

## Installations

## Geopoints