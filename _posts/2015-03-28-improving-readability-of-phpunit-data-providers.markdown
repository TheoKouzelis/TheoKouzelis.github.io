---
layout: post
title: "Improving Readability of PHPUnit Data Providers"
description: "An overview of my naming conventions for my unit tests and data providers."
date: 2015-03-28 19:30:00
categories: php
---
When writing tests I try to describe the test in the function name. I use the format
testDoesSomethingWhenPassedSomething to first describe the assertion and then the context.  

{% highlight php %}
<?php 

class EmailTest extends PHPUnit_Framework_TestCase
{
    public function testReturnsStringWhenPassedCoUkEmail(){
        $result = filter_var("test@test.co.uk", FILTER_VALIDATE_EMAIL);
        $this->assertInternalType('string', $result);
    }

    public function testReturnsStringWhenPassedComEmail(){
        $result = filter_var("test@test.com", FILTER_VALIDATE_EMAIL);
        $this->assertInternalType('string', $result);
    }
}
{% endhighlight %}

When I have many data sets running against the same test I will use the frameworks [data providers](https://phpunit.de/manual/current/en/writing-tests-for-phpunit.html#writing-tests-for-phpunit.data-providers) to 
make the code less verbose.  

{% highlight php %}`
<?php 

class EmailTest extends PHPUnit_Framework_TestCase
{
    public function validEmailProvider(){
        return [
            ['test@test.co.uk'],
            ['test@test.com'],
            ['@invalid@test.io']
        ];
    }

    /**
     * @dataProvider validEmailProvider
     */
    public function testReturnsString($email){
        $result = filter_var($email, FILTER_VALIDATE_EMAIL);
        $this->assertInternalType('string', $result);
    }
}
{% endhighlight %}

The problem with this approach is I have now lost the context from my function name and have reduced 
the readability of my tests. If one of these data sets fails PHPUnit will output the following message.  

{% highlight php %}
EmailTest::testReturnsString with data set #2 ('@invalid@test.io')  
Failed asserting that false is of type "string".  
{% endhighlight %}

The #2 in the error message refers to the numeric index of the array returned from data provider 
which the test failed on. I have discovered that I can improve the readability of the data provider 
function and its error messages by creating  associative arrays instead of numeric ones.  

{% highlight php %}
<?php 

class EmailTest extends PHPUnit_Framework_TestCase
{
    public function validEmailProvider(){
        return [
            'whenPassedCoUkEmail' => [
                'email' => 'test@test.co.uk'
            ],
            'whenPassedComEmail' => [
                'email' => 'test@test.com'
            ],
            'whenPassedIoEmail' => [
                'email' => '@invalid@test.io'
            ]
        ];
    }

    /**
     * @dataProvider validEmailProvider
     */
    public function testReturnsString($email){
        $result = filter_var($email, FILTER_VALIDATE_EMAIL);
        $this->assertInternalType('string', $result);
    }
}
{% endhighlight %}

Now when an error occurs we have regained the context of the test back in the error messages.  

{% highlight php %}
EmailTest::testReturnsString with data set "whenPassedIoEmail" ('@invalid@test.io')  
Failed asserting that false is of type "string".  
{% endhighlight %}
