uFlex 2.0.x
=========================
[![Build Status](https://travis-ci.org/ptejada/uFlex.svg?branch=1.0-DEV)](https://travis-ci.org/ptejada/uFlex) [![Code Climate](https://codeclimate.com/github/ptejada/uFlex/badges/gpa.svg)](https://codeclimate.com/github/ptejada/uFlex) [![Stories in Ready](https://badge.waffle.io/ptejada/uflex.svg?label=ready&title=Ready)](http://waffle.io/ptejada/uflex)
A simple all-in-one PHP user Authentication library.
This library is developed, maintained and tested in a **PHP 5.3.x** environment. The UnitTest also runs
on Travis-CI for **PHP 5.4.x** and **PHP 5.5.x**.

The single class file `class.uFlex.php` code can be found on the [Legacy Branch](https://github.com/ptejada/uFlex/tree/legacy)



For more information:

* Check the examples here <http://ptejada.com/projects/uFlex/examples/>
* Try the demo in this package and review its source.
* See the common methods documentation here <http://ptejada.com/projects/uFlex/documentation/>
* For the full API documentation go to <http://ptejada.com/docs/uFlex/namespaces/ptejada.uFlex.html>

Getting Started
=========================
* [Upgrading from 1.0.x versions](#upgrading-from-1.0.x-versions)
* [Including it in your project](#including-it-in-your-project)
* [Configuring the User object](#configuring-the-user-object)
* [Understanding Collections](#understanding-collections)
* [Using the Session](#using-the-session)
* [Extending the User class](#extending-the-user-class)
* [Contributing](#contributing)

## Upgrading from 1.0.x versions

Before updating you will need to run a SQL upgrade script. Make sure you backup your database before running
the upgrade script. Refer to the DB directory <https://github.com/ptejada/uFlex/tree/master/db>

If not using Composer instead of including a PHP class you will include the `autoload.php` script in
your application which will auto include the library classes as required.

If using composer then just include the `vendor/autoload.php` in your application if it has not already
been included.

## Including it in your project

Use [Composer](https://getcomposer.org/) to add `ptejada/uflex` as a dependency. Note the casing on `uflex`,
all lowercase. Example:
```
{
    "require": {
        "ptejada/uflex": "~2.0"
    }
}
```

If you are already using other [Composer](https://getcomposer.org/) dependencies then you are probably already including
the `vendor/autoload.php` script in your project. If this this is your first composer dependency then you must include
`vendor/autoload.php` in your project. Example:

```php
<?php
require_once 'vendor/autoload.php';
```

If not using Composer then clone this repository in your project. Use the `autoload.php` script to include the library
in your project.

## Configuring the User object

When the `User` class is instantiated not much happens, the session is not initialized nor a DB connection is established.
This is to allow the class to be configured. Once configured the `start()` method must be call in order for the user
authentication process to start. For Example:

```php
<?php
    include_once 'vendor/autoload.php';

    //Instantiate the User object
    $user = new ptejada\uFlex\User();

    // Add database credentials
    $user->config->database->host       = 'localhost';
    $user->config->database->user       = 'test';
    $user->config->database->password   = 'test';
    $user->config->database->name       = 'uflex_test'; //Database name

    // OR if in your project you already have a PDO connection
    // $user->config->database->pdo = $existingPDO;

    /*
     * You can update any customizable property of the class before starting the object
     * construction process
     */

    //Start object construction
    $user->start();
?>
```
It is preferable that a configuration file like the one above is created per project. This way you can use the configuration
file to provide a pre-configured `User` instance to any PHP script in your project.

Alternatively you could create your own class which configures and start the `User` object for you. Example:

```php
<?php
    
    class MyUser extends ptejada\uFlex\User {
        public function __construct()
        {
            parent::__construct();
            
            //Add database credentials
            $this->config->database->host = 'localhost';
            $this->config->database->user = 'test';
            $this->config->database->password = 'test';
            $this->config->database->name = 'uflex_test'; //Database name

            // Start object construction
            $this->start();
        }
    }
?>
```

Below is an excerpt from the PHP class file which lists the customizable `config` properties you could change prior to calling
`start()` on a `User` instance. Note: the `config` property is a `Collection` instance:

```php
    'cookieTime'      => '30',
    'cookieName'      => 'auto',
    'cookiePath'      => '/',
    'cookieHost'      => false,
    'userTableName'   => 'users',
    'userSession'     => 'userData',
    'userDefaultData' => array(
        'Username' => 'Guess',
        'ID'  => 0,
        'Password' => 0,
    ),
    'database' => array(
        'host'     => 'localhost',
        'name'     => '',
        'user'     => '',
        'password' => '',
        'dsn'      => '',
        'pdo'      => '',
    )
```

## Understanding Collections

A `Collection` is an object representation of an array. `Collection`s have many uses throughout this project and are
easy to use. What a `Collection` does for us is handle the errors for undefined indexes and streamline our code.

Consider this example working with plain arrays:

```php
<?php
    $data = array(
        'name' => 'pablo',
    );

    if (isset($data['quote']) && $data['quote'])
    {
        echo $data['name'] . "'s quote is: " . $data['quote'];
    }
    else
    {
        echo $data['name'] . " has no quote";
    }
?>
```

Here is the same code using a `Collection`:

```php
<?php
    $data = ptejada\uFlex\Collection(array(
        'name' => 'pablo',
    ));

    if ($data->quote)
    {
        echo  "{$data->name}'s quote is:  $data->quote";
    }
    else
    {
        echo "{$data->name} has no quote";
    }
?>
```

For more information check the [API Documentation][Collection] for the `Collection` class.

## Using the Session

The `User` object provides an easy interface to manage the PHP session through its `session` property which is an 
instance of the `Session` class. By default the `User` class manages the `userData` namespace in the PHP super global 
`$_SESSION` but this is configurable by setting `config->userSession` before the `User` object is started. This is very 
convenient because it lets the `User` class use the PHP session without interfering with other software that might also 
use the PHP session.

The `Session` class is just an extended `Collection` so it works like any other collection. The only difference is a few
additional methods and the fact that it is a linked collection meaning that any changes made in the object will be
reflected on the global `$_SESSION` and thus automatically saved to the PHP session.

Consider the following code and its output to give you a better idea of how everything works together:

```php
<?php

    $user = new ptejada\uFlex\User();

    // Change the session namespace
    $user->config->userSession = 'myUser';

    $user->start();

    // Shows session right after the User object has been started
    print_r($_SESSION);

    // Stores something in the session
    $user->session->myThing = 'my thing goes here';

    // Shows the session with the content of the myThing property
    print_r($_SESSION);

    // Stores something in the PHP session outside of the namespace scope managed by the User class
    $_SESSION['other'] = 'other information stored here';

    print_r($_SESSION);

    // Only destroys the session namespace managed by the User Class
    $user->session->destroy();

    print_r($_SESSION);

?>
```

Here is the output of the previous code:

```
Array
(
    [myUser] => Array
        (
            [data] => Array
                (
                    [Username] => Guess
                    [ID] => 0
                    [Password] => 0
                )

        )
)

Array
(
    [myUser] => Array
        (
            [data] => Array
                (
                    [Username] => Guess
                    [ID] => 0
                    [Password] => 0
                )

            [myThing] => my thing goes here
        )
)

Array
(
    [myUser] => Array
        (
            [data] => Array
                (
                    [Username] => Guess
                    [ID] => 0
                    [Password] => 0
                )

            [myThing] => my thing goes here
        )

    [other] => other information stored here
)

Array
(
    [other] => other information stored here
)

```

The `Session` class can be use for other aspects of your application as well. For example to manage the entire PHP
session you could do so by making an instance of the `Session` class without arguments: `new ptejada\uFlex\Session()`

For more information on the `Session` class refer to the [API Documentation][Session]

## Extending the User class

In PHP you area able extend classes just like in any object oriented programming language. Therefore you could extend
the `User` class functionality by adding your methods or modifications without having to modify the class file itself. 
You just have create a new PHP class that extends the original `User` class:

```php
<php
    class User extends ptejada\uFlex\User{
        /*
         * Add your default properties values
         * such as database connection credentials
         * user default information
         * Or cookie preferences
         */

        /*
         * Create your own methods
         */
        function updateAvatar(){}

        function linkOpeniD(){}
    }
?>
```

## Contributing

1. Fork it!
2. Create your feature branch: `git checkout -b new-feature`
3. Commit your changes: `git commit -am 'Improved X feature'`
4. Push to the branch: `git push origin new-feature`
5. Submit a pull request from GitHub

[Collection]: http://ptejada.com/docs/uFlex/classes/ptejada.uFlex.Collection.html
[Session]: http://ptejada.com/docs/uFlex/classes/ptejada.uFlex.Session.html
