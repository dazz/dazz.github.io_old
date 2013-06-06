---
layout: post
category : php
tagline: "Webtesting the oauth2 lib"
tags : [testing, oauth2, package, tutorial]
published: false
---
{% include JB/setup %}

Currently I'm writing on an [Hypermedia API](https://www.google.de/search?q=hypermedia+api) .We used a PHP OAuth2 server implementation that integrates into [Silex](http://silex.sensiolabs.org/), a microframework build with Symfony components.

In order to be sure that I did not break anything that should work, because I changed some things and added others for our usecases to work with the [bshaffer/oauth2-server-php](https://github.com/bshaffer/oauth2-server-php) library I added webtests.

The [Symfony HTTP Foundation](http://symfony.com/doc/current/components/http_foundation/index.html) component wraps the http request and response which is used and mocked by [WebTestCase](http://symfony.com/doc/current/book/testing.html#working-with-the-test-client) class which extends the \PHPUnit_Framework_TestCase


### The issue

* ... in this case was that the bridge which integrated the oauth2-server library into Silex extends the `Request` class and expects every Request to be an instance of type `OAuth2\HttpFoundationBridge\Request`. Which meant that not one of the resources behind the oauth2-server would be testable

{% highlight php startinline linenos=table %}
use Symfony\Component\HttpFoundation\Request;
// ...
protected function filterRequest(DomRequest $request)
{
    $httpRequest = Request::create($request->getUri(), $request->getMethod(), ...);
    // ...
}
{% endhighlight %}

As much as Symfony advertises dependency injection they did not use that pattern in the case of the WebTestCase Client implementations.


### My easy solution

* So I took the easy way,
 * extended Silex WebTestCase and Client
 * created instances of `OAuth2\HttpFoundationBridge\Request` where needed

* I also included the possibility to add HTTP_Authorization header as OAuth2 specifies it for POST requests


### The finish

* The last step after I got it working was to split my application specific code from the code that was needed to run the webtests
* pull the code into its own repository
* add a `composer.json`

{% highlight json %}
{
  "name": "dazz/oauth2-server-httpfoundation-webtest",
  "description":"A bridge for webtesting oauth2-server-php with bshaffer/oauth2-server-httpfoundation-bridge and silex",
  "keywords":["oauth","oauth2","auth","httpfoundation", "test", "webtest"],
  "type":"library",
  "license":"MIT",
  "authors":[
    {
      "name":"Hakin Dazs",
    }
  ],
  "homepage": "http://github.com/dazz/oauth2-server-httpfoundation-webtest",
  "require":{
    "php":">=5.3.0",
    "bshaffer/oauth2-server-httpfoundation-bridge": "dev-develop"
  }
}
{% endhighlight %}

* add a README.md

#### How to install

{% highlight json %}
{
  "minimum-stability": "dev",
  "require-dev": {
    "dazz/oauth2-server-httpfoundation-webtest":"dev-master"
  }
}
{% endhighlight %}

* create an account on packagist
* add the hook from packagist in the repository preferences
* add the repository on packagist
* fin :)

### Links

* [github languages](https://github.com/languages)
* https://packagist.org/packages/dazz/oauth2-server-httpfoundation-webtest
* https://github.com/dazz/oauth2-server-httpfoundation-webtest
