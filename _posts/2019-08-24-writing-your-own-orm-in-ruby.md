---
title: "Writing your own ORM in Ruby"
date: "2019-08-24"
layout: post
---

TLDR: [https://github.com/ousmanedev/lite_record](https://github.com/ousmanedev/lite_record)

In this post, we're going to write a basic [ORM](https://en.wikipedia.org/wiki/Object-relational_mapping) (Object-Relational Mapper) in Ruby. If you don't know what an ORM is, or never used one before, I recommend reading this [article](https://blog.bitsrc.io/what-is-an-orm-and-why-you-should-use-it-b2b6f75f5e2a) before. Our ORM will implement the [Active Record](https://en.wikipedia.org/wiki/Active_record_pattern) pattern. This post assumes you know about Ruby, Gems, Bundler and SQL.

## Getting ready

Our ORM, let's call it **LiteRecord** , will be very basic and will only support SQLite3, but will be good enough for you to understand how an ORM works under the cover and from there, you can build a more powerful one.

### Ruby Gem

We will package our code into a gem, to make it easy to install in other projects. Go ahead and run:
```bash
$ bundle gem lite_record
```

We now have a folder (lite_record) where we'll put our ORM code.

### SQLite 3

Run `sqlite3` in your terminal, to make sure you have SQLite3 installed on your machine. If you don't see a prompt, then you'll need to install SQLite3 first ([See how to here](https://mislav.net/rails/install-sqlite3/)). Now, let's install the SQLite3 gem to be able to use SQLite3 from our ruby code. Open the Gemspec file (lite_record.gemspec) inside the lite_record folder, and add this:

spec.add_development_dependency "sqlite3"

Run `bundle install` to install the Gem. We will need a database sample for testing our work, during development. In yout terminal (from the lite_record folder), run:
```bash
$ sqlite3
$ .open test.db;
$ create users(id integer primary key, name varchar(15), email varchar(30));
```

You just created a database (test.db) that contains one table (users).

Now that we're all setup, let's get to the fun part.

## Configuration

LiteRecord needs to know which database it will work with. Open `lib/lite_record.rb` and add the following code in there:

{% gist 924a331fe9eca814d3efabf86632d4b1 %}

In this code, we add a method (configure) to the LiteRecord module. That method receives a path to a SQLite3 database, and then creates and store a connection to that database. That will be very useful to us, later. This is an example of configuration:

{% gist c425e9f10567b36c2a06d8b290ada5b7 %}

## The Base model

Under `lib/lite_record/`, create a file and name it `base.rb`.  In this file, we will create the base model class _LiteRecord::Base_. This class will provides the methods and attributes needed by a typical model to interact with a specific table. Every model classes in a project, will have to extend _LiteRecord::Base_.

{% gist a763cd6f15ef45e88347ae4a8f2d6a83 %}

In this class, we create the constant _DB_, which contains the connection instance created in the configuration step. Also notice, the _attr_accessor: table_ line.O On this line, we allow the model class to specify its mapping table name. This is an example of a model using LiteRecord:

{% gist 1d5880a6bfbe2b0cd637b18a09ad33b5 %}

## Creating records

We want a method that takes a hash of values and create a new record in the database. The hash keys will be the table columns names, and the values will be the new record values. Update _lib/lite_record./base.rb_, with the following code

{% gist f496f1cb8fcfbb23d6fcb19ebca0b432 %}

The _table_columns_ method uses the SQLite3 table_info pragma, to retrieve and return the table columns list without the id column.

The _convert_ method takes a value and transform it into a suitable format for SQL. If the parameter is a _string_, it puts it inside _quotes_ and if it's _nil_, it replaces it will _null_.

We execute a SQL insert statement to create the record, and the use _SELECT last_insert_row_id_ to retrieve the newly created record id.

At the end we return a new instance of the class with the columns values as attributes.

{% gist 85ef9f9cec12e41fd24b0a15b802044f %}

## Finding records

We want to retrieve a record (as an object) from the database with its id. Let's update _lib/lite_record./base.rb_, with a _find_ class method at line 20

{% gist f800c79987a4c506c99b54cfe37c7ee5 %}

It queries the database with the given id and instantiate a new model object with the query results (which will be a hash).

{% gist 97e2a12296c783b25eae15e5ffbe46e2 %}

## Counting records

We want to know the total number of records in the table. Let's update _lib/lite_record./base.rb_, with a _count_ class method at line 24.

{% gist 37c26ea8dca5aab8067bb1f2030d9637 %}

The _count_ method simply runs a _SELECT count_ query and returns the result (which will be an integer).

{% gist 9a9d9dfa5316b7fb8410ce3fa28d7f9c %}

## Saving Objects

We have a model object, that exist in the database or not. When calling the save method, a new record should be created in the database if it didn't exist, otherwise the existing record should be updated with the new values. Let's update the base class, with a _save_ method at line 58.

{% gist a344169cc9a6d37e89d98788c4fef6d1 %}

First notice the [] and []= methods. These methods make it easy to read and write the object attributes hash values.

The _save_ methods first check if the model object has an id attributes. If it doesn't have one, that means the record hasn't been created yet, thus we call the create class method, otherwise we execute a _UPDATE table_ query to update the record with the same id.

{% gist 2fcdab7fae55a8ccdc6421bc9b0793d3 %}

## Deleting records

We have a model object that represents an existing record in the database, and we want to delete that record. Let's add a destroy method inside our base class, at line 66.

{% gist 56e1bea0b0556ed3ae71c1e8ed8026de %}

In this method, we directly run a DELETE FROM query in the database to delete the record with the same id as the object id attribute.

{% gist 1c26924f63992f3071db81b2b00a5c74 %}

## Packaging

Now we have a very basic ORM, unless it's not usable in a project yet. Open the Gemspec file of our Gem and follow the TODO comments to update the metadata informations such as the Gem description, name...

Now, in your terminal run:
```
$ gem build lite_record.gemspec
$ gem install lite_record-0.1.0.gem
$ gem push
```

The full code for this ORM, is on Github. [Click here](https://github.com/ousmanedev/lite_record).

Thanks for reading and don't hesitate to provide (good or bad) feedbacks.
