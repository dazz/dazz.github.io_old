---
layout: post
category : php
tagline: "When no business is around"
tags : [testing, bdd, behat, architecture, tutorial]
published: true
excerpt: "How I came to use Behavoiur Driven Development without using BDD tool behat"

---
{% include JB/setup %}

Given I worked for the last few month on a nice project. Awesome.

When I was writing unit and acceptance tests I extracted methods that setup the pre-conditions I needed for the tests to run successful. I called them `setupAllDatabases` for example (which is no problem as they run with sqlight in memory) and `requestProjects()` to reuse them in other tests.

And when I heard a talk at the [Berlin PHP usergroup](http://www.bephpug.de) from [Nikolas Martens](http://blog.rtens.org/) about "[BDD to the point (pdf)](http://www.bephpug.de/folien/2013-05-07_BDD_to_the_point-Nikoas_Martens.pdf)" and he showed a very simple way to implement a test environment that lets me write testcases the BDD way.

Then I was intrigued. This sounds like a clean way to structure the testcases.

## The pre-face

What Nicolas states in his talk is that BDD is not about tools, but about communication and the understanding of two kinds of groups (business & devs) that initially do not understand each others language easy.

[Behat](http://behat.org/) runs a PHP implementation of [Gherkin](https://github.com/cucumber/cucumber/wiki/Gherkin). The Gherkin language defines a [ubiquitous Domain language](https://en.wikipedia.org/wiki/Behavior-driven_development#Specification_as_a_ubiquitous_language) looking like this:

    As a [role] I want [feature] so that [benefit]

and

    Given [I have x] When [I visit page y] Then [I expect a result z]

that is fairly easy to understands.

**But** as there are no business users involved in the development of the project I am working on it would just bring _"one more tool"_ to the environment that I would have to care about. Which means no outside reason to get Behat in the project.

**But** the domain language makes the tests beautiful. And I just have a thing for beautiful code.

## How will it work?

On the next day after the talk I looked at my tests and saw that it was simple enough to let the testcases be speaking.

I created a `Given`, `When` and `Then` class. The `setupAllDatabases`  function was refactored to `iHaveSetupAllDatabases` and moved to the `Given` class (and all other functions creating pre-conditions equally). All methods that execute the behaviour went to the `When` class and all assertions to `Then` class.

The classes `Given`, `When` and `Then` all extend a `Behaviour` class so I can have easy construction and additional have the benefit that behaviours that live there can be called from `Given` and `When`.

All three behaviour classes are instantated in the Testcase `setUp` function. They get the DIC and additionally the testcase instance injected as constructor parameter so that PHPUnit assertions can be executed in the `Then` instance.

## The test, what it looks like

{% highlight php startinline linenos=table %}
class ProjectControllerTest extends Testcase
{
    /**
     * In order to see all projects
     * As a user
     * I want to request projects and see some
     */
    public function testGetProjects()
    {
        $client = $this->createClient();

        $this->given->iHaveSetupAllDatabases();
        $this->given->iHaveAUser();

        $this->when->iVisitProjects($client);

        $this->then->theResponseStatusShouldBeOk($client);
        $this->then->theResponseShouldContainProjects($client);
    }
}
{% endhighlight %}

### Testcase

{% highlight php startinline linenos=table %}
class TestCase extends \PHPUnit_Framework_TestCase
{
    /** @var \Silex\Application
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

### The root class `Behaviour`

{% highlight php startinline linenos=table %}
use Silex\Application;
use PHPUnit_Framework_TestCase as TestCase;

class Behaviour
{

    /** @var Application $app */
    protected $app;


    /** @var \PHPUnit_Framework_TestCase */
    protected $phpunit;

    /**
     * @param Application $app
     * @param TestCase $test
     */
    public function __construct(Application $app, TestCase $test)
    {
        $this->app = $app;
        $this->test = $test;
    }

    // here we put all functions that can be called
    // by given, when and then
}
{% endhighlight %}

### Given

{% highlight php startinline linenos=table %}
class Given extends Behaviour
{
    /**
     * Sets up all databases and tables
     */
    public function iHaveSetupAllDatabases()
    {
        // ...
    }

    /**
     * Inserts new User in database
     *
     * @return User
     */
    public function iHaveAUser()
    {
        // ...
    }
}
{% endhighlight %}

### When

{% highlight php startinline linenos=table %}
class When extends Behaviour
{
    /**
     * Send request to projects resource
     *
     * @param \Symfony\Component\HttpKernel\Client $client
     */
    public function iVisitProjects($client)
    {
        // call projects page
    }
}
{% endhighlight %}

### Then

{% highlight php startinline linenos=table %}
class Then extends Behaviour
{
    /**
     * Assert that the response status code is 2xx
     *
     * @param \Symfony\Component\HttpKernel\Client $client
     */
    public function theResponseStatusShouldBeOk($client)
    {
        $this->test->assertTrue(
            $client->getResponse()->isSuccessful()
        );
    }

    // ...
}
{% endhighlight %}

## Links

* [Dan North: WHAT’S IN A STORY?](http://dannorth.net/whats-in-a-story/)
* [Dan North: BDD IS LIKE TDD IF…](http://dannorth.net/2012/05/31/bdd-is-like-tdd-if/)