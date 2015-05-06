---
layout: post
title: "DIY vs Framework Mocks"
description: "Comparing creating test mocks using a framework/package or creating them your self."
date: 2015-05-06 01:00:00
categories: php
---
Since learning to test applications I have always used packages like [Mockery](https://github.com/padraic/mockery) or 
[PHPUnit](https://phpunit.de/manual/current/en/test-doubles.html#test-doubles.mock-objects) to create mocks objects. I 
have always found mocking to be a complicated subject and find my tests which involve mocks can be hard to read.  

I have never questioned if there was a more basic way of mocking until I watched [episode 24 of Clean Code](https://cleancoders.com/episode/clean-code-episode-23-p2/show). In this 
episode [Uncle Bob](https://twitter.com/unclebobmartin) demonstrates creating mock objects by extending the original class. This episode prompted me to 
test which technique I prefer for a small and simple project.  

In my example I have written a Greeter class that will return a different greeting depending on the time of day. 
“Good morning” between 5am and 12pm, “Good afternoon” between 12pm and 5pm and “Good evening” for the rest of the 
day. To predict the returned greeting we are going to have to mock out the Greeter class's dependency on the 
DateTime object.  

Writing a mock for the DateTime object is easy. Extend the class overriding any methods you wish to mock. Then store 
all arguments passed to the method in a public variable. Finally make the method return a value of another 
predefined public variable.  

{% highlight php %}
<?php

namespace Kouz\Mocks;

use DateTime;

class DateTimeMock extends DateTime
{
    public $formatReturn = 1;
    public $formatHistory = [];
    
    public function format($format)
    {
        $this->formatHistory[] = $format;
        
        return $this->formatReturn;
    }
}
{% endhighlight %}

In the test we setup up by;  
1. Creating the DateTimeMock.  
2. Setting the return value.   
3. Passing the mock to the class we are testing.   

Then call the greetUser method.   

Then we assert we got;  
1. The expected greeting.  
2. The Greeter called the mock the correct amount of times.  
3. he Greeter passed the mock the expected argument.  

{% highlight php %}
    /**
     * @dataProvider greetingAndHourProvider
     */
    public function testGreetUserReturnsExpectedGreetingWithDiyMock($expectedGreeting, $hour)
    {
        $dateTimeMock = new DateTimeMock();
        $dateTimeMock->formatReturn = $hour;
        $this->greeter->setDateTime($dateTimeMock);
        
        $actualGreeting = $this->greeter->greetUser();
        
        $this->assertEquals($expectedGreeting, $actualGreeting);
        $this->assertCount(1, $dateTimeMock->formatHistory);
        $this->assertEquals('H', $dateTimeMock->formatHistory[0]);
    }
{% endhighlight %}

Here is the same test using PHPUnits mocking class.

{% highlight php %}
    /**
     * @dataProvider greetingAndHourProvider
     */
    public function testGreetUserReturnsExpectedGreetingWithFrameworkMock($expectedGreeting, $hour)
    {
        $dateTimeMock = $this->getMockBuilder('DateTime')
                             ->setMethods(['format'])
                             ->getMock();
                             
        $dateTimeMock->expects($this->once())
                     ->method('format')
                     ->with($this->equalTo('H'))
                     ->willReturn($hour);
              
        $this->greeter->setDateTime($dateTimeMock);
        
        $actualGreeting = $this->greeter->greetUser();
        
        $this->assertEquals($expectedGreeting, $actualGreeting);
    }
{% endhighlight %}

The test code to me looks simpler in the first example, it also follows the three A's of testing, Arrange, Act and 
Assert. When using mocking frameworks I feel the syntax forces us to muddle our assertions in with our arrangements.  

Over large projects maintaining lots of mock classes might become a chore. But in this example I feel creating your 
own mocks increases readability.  

To see the full example please visit the [project on Github](https://github.com/TheoKouzelis/diy-vs-framework-mocks).
