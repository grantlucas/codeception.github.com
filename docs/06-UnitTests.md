---
layout: doc
title: Unit Tests - Codeception - Documentation
---


# Unit Tests

Codeception uses PHPUnit as a backend for running tests. Thus, any PHPUnit test can be added to Codeception test suite and then executed.
If you ever wrote a PHPUnit test then do it just as you did before. 
Codeception adds some nice helpers to simplify common tasks.

The basics of unit tests are skipped here, instead you will get a basic knowledge of what features Codeception adds to unit tests.

__To say it again: you don't need to install PHPUnit to run its tests. Codeception can run them too.__

## Creating Test

Codeception have nice generators to simplify test creation.
You can start with generating a classical PHPUnit test extending `\PHPUnit_Framework_TestCase` class.
This can be done by this command:

{% highlight bash %}

$ php codecept.phar generate:phpunit unit Example

{% endhighlight %}

Codeception has its addons to standard unit tests, so let's try them.
We need another command to create Codeception-powered unit tests.

{% highlight bash %}

$ php codecept.phar generate:test unit Example

{% endhighlight %}

Both tests will create a new `ExampleTest` file located in `tests/unit` directory.

A test created by `generate:test` command will look like this:

{% highlight php %}

<?php
use Codeception\Util\Stub;

class ExampleTest extends \Codeception\TestCase\Test
{
   /**
    * @var UnitTester
    */
    protected $tester;

    // executed before each test
    protected function _before()
    {
    }

    // executed after each test
    protected function _after()
    {
    }
}
?>

{% endhighlight %}

This class has predefined `_before` and `_after` methods to start with. You can use them to create a tested object before each test, and destroy it afterwards.

As you see, unlike in PHPUnit, `setUp` and `tearDown` methods are replaced with their aliases: `_before`, `_after`.

The actual `setUp` and `tearDown` were implemented by parent class `\Codeception\TestCase\Test` and set up the UnitTester class to have all the cool actions from Cept-files to be run as a part of unit tests. Just like in acceptance and functional tests you can choose the proper modules for `UnitTester` class in `unit.suite.yml` configuration file.


{% highlight yaml %}

# Codeception Test Suite Configuration

# suite for unit (internal) tests.
class_name: UnitTester
modules:
    enabled: [UnitHelper, Asserts]

{% endhighlight %}

### Classical Unit Testing

Unit tests in Codeception are written in absolutely the same way as it is done in PHPUnit:

{% highlight php %}

<?php
class UserTest extends \Codeception\TestCase\Test
{
    public function testValidation()
    {
        $this->user = User::create();

        $this->user->username = null;
        $this->assertFalse($user->validate(['username']); 

        $user->username = 'toolooooongnaaaaaaameeee';
        $this->assertFalse($user->validate(['username']);         

        $user->username = 'davert';
        $this->assertTrue($user->validate(['username']));           
    }
}
?>

{% endhighlight %}

### BDD Specification Testing

When writing tests you should prepare them to constant changes in your application. Tests should be easy to read and maintain. If a specification to your application is changed, your tests should be updated as well. If you don't have a convention inside your team on documenting tests, you will have issues figuring out what tests were affected by introduction of a new feature.

That's why it's pretty important not just to cover your application with unit tests, but make unit tests self-explanatory. We do this for scenario-driven acceptance and functional tests, and we should do this for unit and integration tests as well.

For this case we have a stand-alone project [Specify](https://github.com/Codeception/Specify) (which is included in phar package) for writing specifications inside unit tests.

{% highlight php %}

<?php
class UserTest extends \Codeception\TestCase\Test
{
    use \Codeception\Specify;

    public function testValidation()
    {
        $this->user = User::create();

        $this->specify("username is required", function() {
            $this->user->username = null;
            $this->assertFalse($user->validate(['username']); 
        });

        $this->specify("username is too long", function() {
            $user->username = 'toolooooongnaaaaaaameeee';
            $this->assertFalse($user->validate(['username']);         
        });

        $this->specify("username is ok", function() {
            $user->username = 'davert';
            $this->assertTrue($user->validate(['username']));           
        });     
    }
}
?>        

{% endhighlight %}

Using `specify` codeblocks you can describe any piece of test. This makes tests much cleaner and understandable for everyone in your team.

Code inside `specify` blocks is isolated. In the example above any change to `$this->user` (as any other object property), will not be reflected in other code blocks.

Also you may add [Codeception\Verify](https://github.com/Codeception/Verify) for BDD-style assertions. This tiny library adds more readable assertions, which is quite nice, if you are always confused of which argument in `assert` calls is expected and which one is actual.

{% highlight php %}

<?php
verify($user->getName())->equals('john');
?>

{% endhighlight %}

## Using Modules

As in scenario-driven functional or acceptance tests you can access Actor class methods. If you write integration tests, it may be useful to include `Db` module for database testing. 

{% highlight yaml %}

# Codeception Test Suite Configuration

# suite for unit (internal) tests.
class_name: UnitTester
modules:
    enabled: [Db, UnitHelper]

{% endhighlight %}

To access UnitTester methods you can use `UnitTester` property in a test.

### Testing Database

Let's see how you can do some database testing:

{% highlight php %}

<?php
function testSavingUser()
{
    $user = new User();
    $user->setName('Miles');
    $user->setSurname('Davis');
    $user->save();
    $this->assertEquals('Miles Davis', $user->getFullName());
    $this->tester->seeInDatabase('users', array('name' => 'Miles', 'surname' => 'Davis'));
}
?>

{% endhighlight %}

Database will be cleaned and populated after each test, as it happens for acceptance and functional tests.
If it's not your required behavior, please change the settings of `Db` module for the current suite.

### Accessing Module

Codeception allows you to access properties and methods of all modules defined for this suite. Unlike using the UnitTester class for this purpose, using module directly grants you access to all public properties of that module.

For example, if you use `Symfony2` module, here is the way you can access Symfony container:

{% highlight php %}

<?php
/**
 * @var Symfony\Component\DependencyInjection\Container
 */
$container = $this->getModule('Symfony2')->container;
?>

{% endhighlight %}

All public variables are listed in references for corresponding modules.

### Cest

Alternatively to testcases extended from `PHPUnit_Framework_TestCase` you may use Codeception-specific Cest format. It does not require to be extended from any other class. All public methods of this class are tests.

The example above can be rewritten in scenario-driven manner like this:

{% highlight php %}

<?php
class UserCest
{
    public function validateUser(UnitTester $t)
    {
        $user = $t->createUser();
        $user->username = null;
        $t->assertFalse($user->validate(['username']); 

        $user->username = 'toolooooongnaaaaaaameeee';
        $t->assertFalse($user->validate(['username']);         

        $user->username = 'davert';
        $t->assertTrue($user->validate(['username']));

        $t->seeInDatabase('users', ['name' => 'Miles', 'surname' => 'Davis']);
    }
}
?>

{% endhighlight %}

For unit testing you may include `Asserts` module, that adds regular assertions to UnitTester which you may access from `$t` variable.

{% highlight yaml %}

# Codeception Test Suite Configuration

# suite for unit (internal) tests.
class_name: UnitTester
modules:
    enabled: [Asserts, Db, UnitHelper]

{% endhighlight %}

[Learn more about Cest format](http://codeception.com/docs/07-AdvancedUsage#Cest-Classes).

### Stubs

Codeception provides a tiny wrapper over PHPUnit mocking framework to create stubs easily. Include `\Codeception\Util\Stub` to start creating dummy objects.

In this example we instantiate object without calling a constructor and replace `getName` method to return value *john*.

{% highlight php %}

<?php
$user = Stub::make('User', ['getName' => 'john']);
$name = $user->getName(); // 'john'
?>

{% endhighlight %}

Stubs are created with PHPUnit's mocking framework. Alternatively you can use [Mockery](https://github.com/padraic/mockery) (with [Mockery module](https://github.com/Codeception/MockeryModule)), [AspectMock](https://github.com/Codeception/AspectMock) or others.

Full reference on Stub util class can be found [here](/docs/reference/Stub).

## Conclusion

PHPUnit tests are first-class citizens in test suites. Whenever you need to write and execute unit tests, you don't need to install PHPUnit, but use a Codeception to execute them. Some nice features can be added to common unit tests by integrating Codeception modules. For the most of unit and integration testing PHPUnit tests are just enough. They are fast and easy to maintain.




* **Next Chapter: [AdvancedUsage >](/docs/07-AdvancedUsage)**
* **Previous Chapter: [< FunctionalTests](/docs/05-FunctionalTests)**<p>&nbsp;</p><div class="alert alert-warning">Docs are incomplete? Outdated? Or you just found a typo? <a href="https://github.com/Codeception/Codeception/tree/2.0/docs">Help us to improve documentation. Edit it on GitHub</a></div>