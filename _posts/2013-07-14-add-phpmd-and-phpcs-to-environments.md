---
layout: post
category : php
tagline: "Run the tools. Always."
tags : [phpstorm, phpmd, phpcs]
published: true
excerpt: ""

---
{% include JB/setup %}

So clean code matters. To the company, the team, the project and me. A set of rules which define how the code is styled, how long methods are and how sharp the edges feel when you bump them.

## The tools

There are two tools which define and check the adherence of rules. **PHP Mess Detector** and **PHP Code Sniffer**. They bring a good set of rules from where it is easy to start to customize.

There are different times when to let the checks run.

* In the Ide (PhpStorm)
* With every commit
* With every pull request
* With test runs (on travis)

You have your custom rules in the company or even project? Use the [PHP Coding Standard Generator](http://edorian.github.io/php-coding-standard-generator/#phpmd) (thanks edorian).

## In PhpStorm

To integrate the two tools in PhpStorm:

### Add the pathes

* **Settings > PHP > Mess Detector** add the path to phpmd i.e. `/usr/bin/phpmd`

* **Settings > PHP > Code Sniffer** add the path to phpcs i.e. `/usr/bin/phpcs`

### Add the inspections

* **Settings > Inspections > PHP**
* Check **PHP Code Sniffer validation** and
* Check **PHP Mess Detector validation**

There in this deeply buried menu you can also add your own PHP Coding Standard rules.

And voilá the warnings appear where the rules are violated.

## On Travis

Add phpmd in `composer.json`:

{% highlight json startinline linenos=table %}

{
  "require-dev": {
    "phpunit/phpunit": "3.7.*",
    "phpmd/phpmd": "*"
  }
}

{% endhighlight %}

Add the tools to `.travis.yml`:

{% highlight yaml startinline linenos=table %}
language: php

before_script:
  - pear install PHP_CodeSniffer
  - ./composer.phar install --dev --prefer-source --no-interaction

php:
  - 5.3
  - 5.4

script:
  - phpcs --standard=psr2 ./src
  - ./vendor/bin/phpmd src/ text phpmd.xml
  - ./vendor/bin/phpunit
{% endhighlight %}



