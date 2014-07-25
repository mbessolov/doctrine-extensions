Oro Doctrine Extensions
=======================

Table of Contents
-----------------

- [DQL Functions](#dql-functions-list)
    - [Installation](#installation)
    - [Functions Registration](#functions-registration)
        - [Doctrine2](#doctrine2)
        - [Symfony2](#symfony2)
    - [Extendability and Database Support](#extendability-and-database-support)
        - [Architecture](#architecture)
        - [Adding new platform](#adding-new-platform)
        - [Adding new function](#adding-new-function)
- [Field Types](#field-types)

DQL Functions
=============

This library provide set of cross database doctrine DQL functions. Supported databases are MySQL and PostgreSQL.
Available functions:

* `DATE(expr)` - Extract the date part of a date or datetime expression
* `TIME(expr)` - Extract the time portion of the expression passed
* `TIMESTAMP(expr)` - Convert expression to TIMESTAMP

* `DAY(expr)` - Return the day of the month (0-31)
* `DAYOFWEEK(expr)` - Return the weekday index from the date passed
* `DAYOFYEAR(expr)` - Return the day of the year (1-366)
* `HOUR(expr)` - Return the hour from the date passed
* `MINUTE(expr)` - Return the minute from the date passed
* `MONTH(expr)` - Return the month from the date passed
* `QUARTER(expr)` - Return the quarter from the date passed
* `SECOND(expr)` - Return the second from the date passed
* `WEEK(expr)` - Return the week from the date passed
* `YEAR(expr)` - Return the year from the date passed

* `POW(expr, power)` - Return the argument raised to the specified power
* `SIGN(expr)` - Return the sign of the argument

Return a concatenated string
```
GROUP_CONCAT([DISTINCT] expr [,expr ...]
            [ORDER BY {unsigned_integer | col_name | expr}
                [ASC | DESC] [,col_name ...]]
            [SEPARATOR str_val])
```

Installation
------------

Add the following dependency to your composer.json
```json
{
    "require": {
        "orocrm/doctrine-extensions": "dev-master"
    }
}
```
Functions Registration
----------------------

### Doctrine2 

(http://docs.doctrine-project.org/en/latest/cookbook/dql-user-defined-functions.html "DQL User Defined Functions")

```php
<?php
$config = new \Doctrine\ORM\Configuration();
$config->addCustomStringFunction('group_concat', 'Oro\ORM\Query\AST\Functions\String\GroupConcat');
$config->addCustomNumericFunction('hour', 'Oro\ORM\Query\AST\Functions\SimpleFunction');
$config->addCustomDatetimeFunction('date', 'Oro\ORM\Query\AST\Functions\SimpleFunction');

$em = EntityManager::create($dbParams, $config);
```

### Symfony2 

In Symfony2 you can register functions in `config.yml`

```yaml
doctrine:
    orm:
        dql:
            datetime_functions:
                date:         Oro\ORM\Query\AST\Functions\SimpleFunction
                time:         Oro\ORM\Query\AST\Functions\SimpleFunction
                timestamp:    Oro\ORM\Query\AST\Functions\SimpleFunction
            numeric_functions:
                dayofmonth:   Oro\ORM\Query\AST\Functions\SimpleFunction
                dayofweek:    Oro\ORM\Query\AST\Functions\SimpleFunction
                dayofyear:    Oro\ORM\Query\AST\Functions\SimpleFunction
                week:         Oro\ORM\Query\AST\Functions\SimpleFunction
                day:          Oro\ORM\Query\AST\Functions\SimpleFunction
                hour:         Oro\ORM\Query\AST\Functions\SimpleFunction
                minute:       Oro\ORM\Query\AST\Functions\SimpleFunction
                month:        Oro\ORM\Query\AST\Functions\SimpleFunction
                quarter:      Oro\ORM\Query\AST\Functions\SimpleFunction
                second:       Oro\ORM\Query\AST\Functions\SimpleFunction
                year:         Oro\ORM\Query\AST\Functions\SimpleFunction
                sign:         Oro\ORM\Query\AST\Functions\Numeric\Sign
                pow:          Oro\ORM\Query\AST\Functions\Numeric\Pow
            string_functions:
                group_concat: Oro\ORM\Query\AST\Functions\String\GroupConcat
```

Extendability and Database Support
----------------------------------

### Architecture

Most of functions, that require only one ArithmeticPrimary argument may be parsed with `Oro\ORM\Query\AST\Functions\SimpleFunction`.
This class is responsible for parsing function definition and saving parsed data to parameters. It is extended from
`Oro\ORM\Query\AST\Functions\AbstractPlatformAwareFunctionNode`. This layer work with DQL function parsing.
SQL generation is responsibility of platform specific functions, that extends `PlatformFunctionNode`.
`AbstractPlatformAwareFunctionNode` creates appropriate instance of platform function based on current connection Database Platform instance
name and DQL function name.
Platform function classes naming rule is `Oro\ORM\Query\AST\Platform\Functions\$platformName\$functionName`

### Adding new platform
To add support of new platform you just need to create new folder `Oro\ORM\Query\AST\Platform\Functions\$platformName`
and implement required function there according to naming rules

### Adding new function

In case when your function is function with only one ArithmeticPrimary argument you may not create DQL function parser
and use `Oro\ORM\Query\AST\Functions\SimpleFunction` for this. 
Then only platform specific SQL implementation of your function is required.

In case when your are implementing more complex function, like GROUP_CONCAT both DQL parser and SQL implementations are required.

If you want to add new function to this library feel free to fork it and create pull request with your implementation.
Please, remember to update documentation with your new functions. All new functions MUST be implemented for all platforms.

Field Types
===========

This library also consist additional field types:

* `MoneyType`
* `PercentType`
* `ObjectType`
* `ArrayType`

`ObjectType` and `ArrayType` use base64 encoded string to store values in Db instead of storing serialized strings.