---
layout: post
category : php
tagline: "BDT without behat"
tags : [testing, bdd, behat, architecture, tutorial]
---
{% include JB/setup %}

## Setup

```php
class TestCase extends \PHPUnit_Framework_TestCase
{
    protected $app;

    /** @var Given */
    protected $given;

    /** @var When */
    protected $when;

    /** @var Then */
    protected $then;

    /**
     * PHPUnit setUp for setting up the application.
     *
     * Note: Child classes that define a setUp method must call
     * parent::setUp().
     */
    public function setUp()
    {
        $this->app = $this->createApplication();

        $this->given = new Given($this->app, $this);
        $this->when = new When($this->app, $this);
        $this->then = new Then($this->app, $this);
    }

    // ...
}
```

The classes `Given`, `When` and `Then` all extend a `Behaviour` class

```php
class Behaviour
{

    /** @var \Silex\Application $app */
    protected $app;


    /** @var \PHPUnit_Framework_TestCase */
    protected $phpunit;

    /**
     * @param \Silex\Application $app
     * @param \PHPUnit_Framework_TestCase $phpunit
     */
    public function __construct(\Silex\Application $app, \PHPUnit_Framework_TestCase $phpunit)
    {
        $this->app = $app;
        $this->phpunit = $phpunit;
    }

    // here we put all functions that can be call by given, when and then
}
```