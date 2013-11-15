# Ask

Library containing a PHP implementation of the Ask query language.

The implementation includes the domain objects that represent various parts of Ask queries,
as well as serializers and deserializers for these objects.

[![Build Status](https://secure.travis-ci.org/wmde/Ask.png?branch=master)](http://travis-ci.org/wmde/Ask)
[![Coverage Status](https://coveralls.io/repos/wmde/Ask/badge.png?branch=master)](https://coveralls.io/r/wmde/Ask?branch=master)
[![Dependency Status](https://www.versioneye.com/user/projects/5281036c632bacd30e000112/badge.png)](https://www.versioneye.com/user/projects/5281036c632bacd30e000112)

On Packagist:
[![Latest Stable Version](https://poser.pugx.org/ask/ask/version.png)](https://packagist.org/packages/ask/ask)
[![Download count](https://poser.pugx.org/ask/ask/d/total.png)](https://packagist.org/packages/ask/ask)

## Requirements

* PHP 5.3 or later
* [DataValues](https://www.mediawiki.org/wiki/Extension:DataValues) 0.1 or later
* [Serialization](https://github.com/wmde/Serialization/blob/master/README.md) 2.x

## Installation

You can use [Composer](http://getcomposer.org/) to download and install
this package as well as its dependencies. Alternatively you can simply clone
the git repository and take care of loading yourself.

### Composer

To add this package as a local, per-project dependency to your project, simply add a
dependency on `ask/ask` to your project's `composer.json` file.
Here is a minimal example of a `composer.json` file that just defines a dependency on
Ask 1.0:

    {
        "require": {
            "ask/ask": "1.0.*"
        }
    }

### Manual

Get the Ask code, either via git, or some other means. Also get all dependencies.
You can find a list of the dependencies in the "require" section of the composer.json file.
Load all dependencies and the load the Ask library by including its entry point:
Ask.php.

## Structure

The Ask library defines the Ask query language. Its important components are:

* Ask\Language - everything part of the ask language itself

    * Ask\Language\Description - descriptions (aka concepts)
    * Ask\Language\Option - QueryOptions object and its parts
    * Ask\Language\Selection - selection requests
    * Ask\Language\Query.php - the object defining what a query is

### Description

Each query has a single description which specifies which entities match. This is similar to the
WHERE part of an SQL string. There different types of descriptions are listed below. Since several
types of descriptions can be composed out of one or more sub descriptions, tree like structures can
be created.

* Description - abstract base class

    * AnyValue - A description that matches any object
    * Conjunction - Description of a collection of many descriptions, all of which must be satisfied (AND)
    * Disjunction - Description of a collection of many descriptions, at least one of which must be satisfied (OR)
    * SomeProperty - Description of a set of instances that have an attribute with some value that fits another (sub)description
    * ValueDescription - Description of one data value, or of a range of data values

All descriptions reside in the Ask\Language\Description namespace.

### Option

The options a query consist out of are defined by the <code>QueryOptions</code> class. This class
contains limit, offset and sorting options.

Sorting options are defined by the <code>SortOptions</code> class, which contains a list of
<code>SortExpression</code> objects.

All options related classes reside in the Ask\Language\Option namespace.

### Selection

Specifying what information a query should select from matching entities is done via the selection
requests in the query object. Selection requests are thus akin to the SELECT part of an SQL string.
They thus have no effect on which entities match the query and are returned. All types of selection
request implement abstract base class SelectionRequest and can be found in the Ask\Language\Selection
namespace.

## Usage

#### A query for the first hunded entities that are compared

```php
use Ask\Language\Query;
use Ask\Language\Description\AnyValue;
use Ask\Language\Option\QueryOptions;

$myAwesomeQuery = new Query(
    new AnyValue(),
    array(),
    new QueryOptions( 100, 0 )
);
```

#### A query with an offset of 50

```php
$myAwesomeQuery = new Query(
    new AnyValue(),
    array(),
    new QueryOptions( 100, 50 )
);
```

#### A query to get the ''cost'' of the first hundered entities that have a ''cost'' property

This is assuming 'p42' is an identifier for a ''cost'' property.

```php
$awesomePropertyId = new PropertyValue( 'p42' );

$myAwesomeQuery = new Query(
    new SomeProperty( $awesomePropertyId, new AnyValue() ),
    array(
        new PropertySelection( $awesomePropertyId )
    ),
    new QueryOptions( 100, 0 )
);
```

#### A query to get the first hundred entities that have 9000.1 as value for their ''cost'' property.

This is assuming 'p42' is an identifier for a ''cost'' property.

```php
$awesomePropertyId = new PropertyValue( 'p42' );
$someCost = new NumericValue( 9000.1 );

$myAwesomeQuery = new Query(
    new SomeProperty( $awesomePropertyId, new ValueDescription( $someCost ) ),
    array(),
    new QueryOptions( 100, 0 )
);
```

#### A query getting the hundred entities with highest ''cost'', highest ''cost'' first

This is assuming 'p42' is an identifier for a ''cost'' property.

```php
$awesomePropertyId = new PropertyValue( 'p42' );

$myAwesomeQuery = new Query(
    new AnyValue(),
    array(),
    new QueryOptions(
        100,
        0,
        new SortOptions( array(
            new PropertyValueSortExpression( $awesomePropertyId, SortExpression::DESCENDING )
        ) )
    )
);
```

#### A query to get the hundred first entities that have a ''cost'' either equal to 42 or bigger than 9000

This is assuming 'p42' is an identifier for a ''cost'' property.

```php
$awesomePropertyId = new PropertyValue( 'p42' );
$costOf42 = new NumericValue( 42 );
$costOf9000 = new NumericValue( 9000 );

$myAwesomeQuery = new Query(
    new SomeProperty(
        $awesomePropertyId,
        new Disjunction( array(
            new ValueDescription( $costOf42 ),
            new ValueDescription( $costOf9000, ValueDescription::COMP_GRTR ),
        ) )
    ),
    array(),
    new QueryOptions( 100, 0 )
);
```

### Serialization and deserialization

The Ask language objects can all be serialized to a generic format from which the objects can later
be reconstructed. This is done via a set of Serializers/Serializer implementing objects. These
objects turn for instance a Query object into a data structure containing only primitive types and
arrays. This data structure can thus be readily fed to json_enoce, serialize, or the like. The
process of reconstructing the objects from such a serialization is provided by objects implementing
the Deserializers/Deserializer interface.

Serializers can be obtained via an instance of SerializerFactory and deserializers can be obtained
via an instance of DeserializerFactory. You are not allowed to construct these serializers and
deserializers directly yourself or to have any kind of knowledge of them (ie type hinting). These
objects are internal to the Ask library and might change name or structure at any time. All you
are allowed to know when calling $serializerFactory->newQuerySerializer() is that you get back
an instance of Serializers\Serializer.

## Tests

This library comes with a set up PHPUnit tests that cover all non-trivial code. You can run these
tests using the PHPUnit configuration file found in the root directory. The tests can also be run
via TravisCI, as a TravisCI configuration file is also provided in the root directory.

## Authors

Ask has been written by [Jeroen De Dauw](https://www.mediawiki.org/wiki/User:Jeroen_De_Dauw)
as [Wikimedia Germany](https://wikimedia.de) employee for the [Wikidata project](https://wikidata.org/).

## Release notes

### 1.0 (under development)

Initial release with these features:

* PHP implementation of the Ask language core
* Implementation of descriptions, selection requests and sort options initially needed for Wikidata
* Serializers for all implemented Ask language objects
* Deserializers for all implemented Ask language objects

## Links

* [Ask on Packagist](https://packagist.org/packages/ask/ask)
* [Ask on Ohloh](https://www.ohloh.net/p/ask)
* [TravisCI build status](https://travis-ci.org/wmde/Ask)
* [NodeJS implementation of Ask](https://github.com/JeroenDeDauw/AskJS)
