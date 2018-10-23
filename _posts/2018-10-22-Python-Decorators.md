---
layout: post
title:  "Python Decorators"
date:   2018-10-22
excerpt: "One of the must know Python features."
tags: [python]
comments: true
belongsto: posts
---
In this post, I’ll provide a basic introduction to decorators. 

Decorators provide a simple syntax for calling [higher-order](https://en.wikipedia.org/wiki/Higher-order_function) functions. To understand how they work, one must first understand how functions work.
### Functions
For the most basic of purposes, **a function returns values based on given arguments**.

Consider the following function.
{% highlight python %}
def greet_person(person_name):
	return "Hey {}".format(person_name)
{% endhighlight %}
{% highlight python %}
>>> greet_person("Bob") 
'Hey Bob'
{% endhighlight %}

There's nothing fancy about its implementation. It works as intended.
### First class objects
In Python, functions are [first-class](https://en.wikipedia.org/wiki/First-class_citizen) objects. This means that functions can be passed to other functions as if they were objects.

Consider the following example.
{% highlight python %}
def greet_with_hi():
	return "Hi {}".format(person)

def greet_with_hello(person):
	return "Hello {}".format(person)

def greet(greeting_method, person):
	return greeting_method(person)
{% endhighlight %}

Here, *greet_with_hi* and *greet_with_hello* are simple functions which expect a string and returns the appropriate greeting string.
However, *greet* accepts a function as an argument and calls it from within.

{% highlight python %}
>>> greet(greet_with_hi, "Bob")
'Hi Bob'
>>> greet(greet_with_hello, "Charles")
'Hello Charles'
{% endhighlight %}

### Returning functions
Python also supports defining functions locally and returning them.
{% highlight python %}
def get_greeter_function(name, greeting_type = 'hi'):
	def hi_greeter():
		return "Hi {} This is the first greeter function.".format(name)

	def custom_greeter():
		return "{} {} This is the second greeter function.".format(greeting_type, name)

	if greeting_type == 'hi':
		return hi_greeter
	else:
		return custom_greeter
{% endhighlight %}
{% highlight python %}
>>> greet = get_greeter_function("Bob")
>>> greet()
'Hi Bob This is the first greeter function.'
>>> another_greet = get_greeter_function("Charles", "Howdy")
>>> another_greet()
'Howdy Bob This is the second greeter function.'
{% endhighlight %}
Although it seems fairly obvious, it is prudent we note that the local functions cannot be called from outside the scope of the function.
{% highlight python %}
>>> hi_greeter()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'hi_greeter' is not defined
{% endhighlight %}
### Decorators
Now let's decorate a function.
{% highlight python %}
def with_greetings(do_something):
	def wrapped_function():
		print('Hello')
		do_something()
		print('Goodbye')
	return wrapped_function
{% endhighlight %}
{% highlight python %}
>>> def say_message():
...		print("A message.")
...
>>> say_message = with_greetings(say_message)
>>> say_message()
Hello
A message.
Goodbye
{% endhighlight python %}
We've wrapped *say_message()* to display the message along with 'Hello' and 'Goodbye' messages.
### Syntactic sugar
The above method of adding a decorator still feels a bit clunky. This is where the beast that is pythonic decorators comes into play.
{% highlight python %}
def with_greetings(do_something):
	def wrapped_function():
		print('Hello')
		do_something()
		print('Goodbye')
	return wrapped_function

@with_greetings
def do_something():
	print("The cow goes moo.")

@with_greetings
def do_something_else():
	print("Horses neigh")
{% endhighlight %}
This bypasses the cumbersome process of having to reassign functions.
{% highlight python %}
>>> do_something()
Hello
The cow goes moo.
Goodbye
>>> do_something_else()
Hello
Horses neigh
Goodbye
{% endhighlight %}
### Decorating with arguments
It doesn't make any sense to tailor the arguments of *wrapped_function* to the arguments of the function within. Using variadic positional arguments and keyword arguments makes the job easier for us.
{% highlight python %}
def with_greetings(do_something):
	def wrapped_function(*args, **kwargs):
		print('Hello')
		do_something(*args, **kwargs)
		print('Goodbye')
	return wrapped_function
{% endhighlight %}
### Returning values
Now it's just a matter of common sense. The functions tailing the *do_something()* function call would be unreachable if the result of the function was returned as is.
{% highlight python %}
def with_greetings(do_something):
	def wrapped_function(*args, **kwargs):
		# do stuff before
		print('Hello')
		# store the returned value
		ret = do_something(*args, **kwargs)
		# do stuff after
		print('Goodbye')
		# return
		return ret
	return wrapped_function
{% endhighlight %}

### More ground.
I can't think of anything else to add. Hopefully this gives a basic primer on the implementation and usage of decorators.
