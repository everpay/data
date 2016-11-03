# Agile Data

[![Build Status](https://travis-ci.org/atk4/data.png?branch=develop)](https://travis-ci.org/atk4/data)
[![Code Climate](https://codeclimate.com/github/atk4/data/badges/gpa.svg)](https://codeclimate.com/github/atk4/data)
[![StyleCI](https://styleci.io/repos/56442737/shield)](https://styleci.io/repos/56442737)
[![Test Coverage](https://codeclimate.com/github/atk4/data/badges/coverage.svg)](https://codeclimate.com/github/atk4/data/coverage)
[![Version](https://badge.fury.io/gh/atk4%2Fdata.svg)](https://packagist.org/packages/atk4/data)


**PHP Framework for better Business Logic design and scalable database access.**

Agile Data implements various advanced database access patterns such as Active Record, Persistence Mapping, Domain Model, Event sourcing, Actions, DataSets and Query Building in a **practical way** that can be **easily learned**, used in any framework with SQL or NoSQL database and meeting all **enterprise**-specific requirements.

``` php
$m = new Client($db);
echo $m->addCondition('vip', true)
  ->ref('Order')->ref('Line')->action('fx', ['sum', 'total']);
```

Resulting Query:

``` sql
select sum(`price`*`qty`) from `order_line` `O_L` where `order_id` in (
  select `id` from `order` `O` where `client_id` in (
    select `id` from `client` where `vip` = 'Y'
  )
)
```

One of the goals is to keep your entire domain logic away from abstracted data-stores while using the advanced query patterns like JOIN, SUBQUERY, UNION to boost scalability. DataSet solve decade-old ORM problems such as `N+1`. All of that is achieved with a beautiful and consise syntax, that does not rely on extra XML files or annotations. Next snippet illustrates data import, that will automatically map into 3-related records across 2 or more SQL tables:

``` php
$c = new Client($db);
$c->loadBy('name', 'Pear Company');
$c->ref('Invoice')
   ->save(['ref'=>'TBL1', 'due'=>new DateTime('+1 month')])
   ->ref('Lines')->import([
      ['table', 'qty'=>2, 'price'=>10.50],
      ['chair', 'qty'=>10, 'price'=>3.25],
]);
```

Agile Data is not only for SQL databases. It can be used from interpreting Form submission data from $_POST into domain model to RestAPI abstractions and proxying. Zero-configuration implementation for "AuditTrail", "ACL" and "Soft Delete" as well as new features such as "Undo", "Global Scoping" and "Cross-persistence" make your Agile Data code enterprise-ready out of the box.

If you enjoy advanced examples, continue to https://github.com/atk4/data-primer.

## The Gentle Beginning

You must be ready to learn, even if you are familiar with all of the Martin Fowler's works and several different PHP ORM implementations. Various features and patterns are exclusive to Agile Data. 

http://socialcompare.com/en/comparison/php-data-access-libraries-orm-activerecord-persistence

Use Agile Data in your existing PHP Apps to boost scalability, performance, security and clean up your code.

## Essential Links

- Slides from the presentation: [Love and Hate Relationship between ORMs and Query Builders](http://www.slideshare.net/romaninsh/love-and-hate-relationship-between-orm-and-query-builders)" talk.

- (https://gitter.im/atk4/dataset?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

- Agile Data is part of [Agile Toolkit](http://agiletoolkit.org/). See our list of commercial services and extensions for [Agile Data](http://agiletoolkit.org/data).

![Gitter](https://img.shields.io/gitter/room/atk4/data.svg?maxAge=2592000)[![Documentation Status](https://readthedocs.org/projects/agile-data/badge/?version=develop)](http://agile-data.readthedocs.io/en/develop/?badge=latest)[![Stack Overlfow Community](https://img.shields.io/stackexchange/stackoverflow/t/atk4.svg?maxAge=2592000)](http://stackoverflow.com/questions/ask?tags=atk4)[![Discord User forum](https://img.shields.io/badge/discord-User_Forum-green.svg)](https://forum.agiletoolkit.org/c/44)

### Introducing Models

Start by creating a new class that defines your Models. In Agile Data the `Model` class is a lightweight object, to augment your business entities.

``` php
class Client extends \atk4\data\Model {
  public $table = 'client';
  function init() {
    parent::init();
    
    $this->addFields(['name','address']);
    
    $this->hasMany('Project', new Project());
  }
}
```

-   Documentation: http://agile-data.readthedocs.io/en/develop/model.html
-   Examples: https://github.com/atk4/data-primer/tree/master/src

### Introducing Actions

 ![mapping](docs/images/mapping.png)

Anything related to a Model (Field, Condition, Reference) lives is the realm of "Domain Model" inside PHP memory. When you  `save()`, frameworks generates an "Action" that will actually update your SQL table, invoke RestAPI request or write that file to disk.

For SQL-persistence, Actions obect are build using a powerful SQL Query Builder that gives you a window into all the features of Relational databases that many ORM simply choose to "lock away". Actions can either query or change the data and they can work with either SQL "fields" or DomainModel "fields":

![GitHub release](docs/images/action.gif)

### Introducing Expressions

In Agile Data your Domain Model Field (such as `$project['budget']`) can be defined through user-defined SQL expression. Currently only SQL Persistence fully suports expressions. Domain model "fields" that map into expression behave no different to the read-only physical field to the rest of your application.

![GitHub release](docs/images/expression.gif)

### Introducing References

Foreign keys and Relation are bread and butter of RDBMS. While it makes sense in "Persistence", not all databases support Relations.

Agile Data takes a different approach by introducing "References". It allow you to define relationships between Domain Models that can work with non-relational databases, yet allow you to perform various operations such as importing or aggregating fields.

![GitHub release](docs/images/import-field.gif)

### Model Conditions and DataSets

Conditions (or scopes) are rare and optional feature across ORMs but it is one of the most significant features in Agile Data. It allows you to create objects that represent multiple database records without actually loading them.  

Once condition is defined, it's will appear in actions and will also restrict you from adding non-compliant records.

![GitHub release](docs/images/reference-magic.gif)


### Build Reports inside Domain Model

With most frameworks when it comes to serious data aggregation you must make a choice - write in-efficient domain-model code or write RAW SQL query. Agile Data helps you tap into unique features of your DataBase while letting you stay inside Domain Model.

How do we create an efficient query to display total budget from all the projects grouped by client's country while entirely remaining in domain model?

![GitHub release](docs/images/domain-model-reports.gif)

Did you notice that the amount excluded cancelled projects even though we never asked so
explicitly?

### Model-level join

Most ORMs can define define models that only work with a single SQL table. If you have
to store logical entity data into multiple tables - tough luck, you'll have to do
some linking yourself.

Agile Data allow you to define multiple joins right inside your model. As you join()
another table, you will be able to import fields from the joined table. If you
create a new record, data will automatically be distributed into the tables and
records will be linked up correctly.

![GitHub release](docs/images/model-join.gif)

### Deep Model Traversal

Probably one of the best feature of Agile Data is deep traversal. Remember how
your ORM tried to implement varous many-to-many relationships? This is no longer
a problem in Agile Data.

Suppose you want to look at all the countries that have 2-letter name. How many
projects are there from the clients that are located in a country with 2-letter name?

Agile Data can answer with a query or with a result.


![GitHub release](docs/images/deep-traversal.gif)

## Advanced Features

The examples you saw so far are only a small fragment of the possibilities you can
achieve with Agile Data. You now have a new playground where you can design your
business logic around the very powerful concepts.

One of the virtues we value the most in Agile Data is ability to abstract and
add higher level features on our solid foundation.

### Hooks

You now have a domain-level and persistence-level hooks. With a domain-level ones (afterLoad, beforeSave) you get to operate with your field data before or after operation.

On other hand you can utilise persistence-level hooks ('beforeUpdateQuery', 'beforeSelectQuery') and you can interact with a powerful Query Builder to add a few SQL options (insert ignore or calc_found_rows)
if you need.

And guess what - should your model be saved into NoSQL database, the domain-level hooks will be executed, but SQL-specific ones will not.

### Extensions

Most ORMs hard-code features like soft-delete, audit-log, timestamps. In Agile Data the implementation of base model is incredibly lightweight and all the necessary features are added through external objects.

We are still working on our Extension library but we plan to include:

- Audit Log - record all operations in a model (as well as previous field values)
- Undo - revert last few few operations on your model.
- ACL - flexible system to restrict access to certain records, fields or models based on
  permissions of your logged-in user or custom logic.
- Filestore - allow you to work with files inside your model. Files are actually
  stored in S3 (or other) but the references and meta-information remains in the database.
- Soft-Delete, purge and undelete - several strategies, custom fields, permissions.

If you are interested in early access to Extensions, please contact us at
http://agiletoolkit.org/contact

### Performance

If you wonder how those advanced features may impact performance of loading and saving data, there is another pleasant surprise. Loading, saving, iterating and deleting records do not create new in-memory objects:


``` php
foreach($client->ref('Project') as $project) {
    echo $project->get('name')."\n"
}

// $project refers to same object at all times, but $project's active data
// is re-populated on each iteration.
```

Nothing unnecessary is pre-fetched. Only requested columns are queried. Rows are streamed and never ever we will try to squeeze a large collection of IDs into a query!

Agile Data works fast even if you have huge amount of records in the database.

### Security

When ORM promise you "security" they don't really extend it to the cases where you wish to perform a sub-query of a sort. Then you have to deal with RAW query components and glue them together yourself.

Agile Data provides a universal support for Expressions and each expression have supports for `escaping` and `parameters`;

``` php
$c->addCondition($c->expr('length([name]) = []', [20]))
```

This is condition from our deep-traversal demo, where our custom condition
fetches only 2-character long countries. Compare that to the generated query
segment:

``` php
where length(`name`) = :a  [:a=20]
```

First of all - `[name]` is automatically mapped into SQL representation of your name field (in case it's field from a join or a sub-query). Secondly the number `2` is supplied as PDO parameter. And Agile Data takes extra care to join parameters between different expressions that make it into your query.

The final security measure are the Conditions. Once you load your Client, traversing into 'Project' model will imply a condition which will only expose projects of that specific Client.

Even if you perform a multi-row opetation such as `action('update')` or `action('delete')` it will only apply to projects of that client. With the model object you won't be able to create a new project that does NOT belong to loaded client.

Those security measures are there to protect you against human factor.

### Full documentation for Agile Data

If you have missed link to documentation, then its [agile-data.readthedocs.io](http://agile-data.readthedocs.io).

### Getting Started Guides

* [Follow the Quick Start guides](http://agile-data.readthedocs.io/en/develop/quickstart.html)
* [Watch short introduction video on Youtube](https://youtu.be/ZekgUxdPWwc)

## Installing into existing project

Update your `composer.json` with 'require' and 'autoload' sections:

``` json
{
  "type":"project",
  "require":{
    "atk4/data": "^1.0.0",
    "psy/psysh": "*"
  },
  "autoload":{
    "psr-4": {
      "my\\": "src/my/"
    }
  }
}
```

Run `composer update` and create your first business model inside `src/my/Model_User.php`:

``` php
namespace my;
class Model_User extends \atk4\data\Model
{
    public $table = 'user';
    function init()
    {
        parent::init();

        $this->addFields(['email','name','password']);
    }
}
```

Use an existing table name and fields. Next create `console.php` file to start exploring Agile Data:

``` php
<?php
include'vendor/autoload.php';
$db = \atk4\data\Persistence::connect(PDO_DSN, USER, PASS);
$m = new my\Model_User($db);
eval(\Psy\sh());
```

Finally, run `console.php`:

```
$ php console.php
```

Now you can explore. Try typing:

``` php
> $m
> $m->loadBy('email', 'example@example.com')
> $m->get()
> $m->export(['email','name'])
> $m->action('count')
> $m->action('count')->getOne()
```

## Agile Toolkit

Agile Data is part of [Agile Toolkit - PHP UI Framework](http://agiletoolkit.org). If you like
this project, you should also look into:

- [DSQL](https://github.com/atk4/dsql) - [![GitHub release](https://img.shields.io/github/release/atk4/dsql.svg?maxAge=2592000)]()
- [Agile Core](https://github.com/atk4/core) - [![GitHub release](https://img.shields.io/github/release/atk4/core.svg?maxAge=2592000)]()


## Help us make Agile Data better!!

We wish to take on your feedback and improve Agile Data further. Here is how you can connect with developer team:

- chat with us on [Gitter](https://gitter.im/atk4/data) and ask your questions directly.
- ask or post suggestions on our forum [https://forum.agiletoolkit.org](https://forum.agiletoolkit.org)
- **share Agile Data with your friends**, we need more people to use it. Blog. Tweet. Share.
- work on some of the tickets marked with [help wanted](https://github.com/atk4/data/labels/help%20wanted) tag.

See [www.agiletoolkit.org](http://www.agiletoolkit.org/) for more frameworks and libraries that can make your PHP Web Application even more efficient.

## Roadmap

Follow pull-request history and activity of repository to see what's going on.

```
1.1   Add support for derived models (unions).
1.x   Add support for 3rd party vendor implementations.
1.x   Add support for MongoDB.
1.x   Add support and docs for Validators.
```

## Past Updates
* 20 Jul: Release of 1.0 with a new QuickStart guide
* 15 Jul: Rewrote README preparing for our first BETA release
* 05 Jul: Released 0.5 Expressions, Conditions, Relations
* 28 Jun: Released 0.4 join support for SQL and Array
* 24 Jun: Released 0.3 with general improvements
* 17 Jun: Finally shipping 0.2: With good starting support of SQL and Array
* 29 May: Finished implementation of core logic for Business Model
* 11 May: Released 0.1: Implemented code climate, test coverage and travis
* 06 May: Revamped the concept, updated video and made it simpler
* 22 Apr: Finalized concept, created presentation slides.
* 17 Apr: Started working on concept draft (in wiki)
* 14 Apr: [Posted my concept on Reddit](https://www.reddit.com/r/PHP/comments/4f2epw/reinventing_the_faulty_orm_concept_subqueries/)
