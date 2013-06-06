---
layout: post
category : php
tagline: "When no management is around"
tags : [testing, bdd, behat, architecture, tutorial]
published: false
---
{% include JB/setup %}

## Setup

{% highlight php startinline linenos=table %}
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
{% endhighlight %}

The classes `Given`, `When` and `Then` all extend a `Behaviour` class

{% highlight php startinline linenos=table %}
use Silex\Application;
use PHPUnit_Framework_TestCase as TestCase;

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
    public function __construct(Application $app, TestCase $phpunit)
    {
        $this->app = $app;
        $this->phpunit = $phpunit;
    }

    // here we put all functions that can be call by given, when and then
}
{% endhighlight %}