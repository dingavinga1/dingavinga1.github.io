---
title: Polymorphism Hidden in Plain Sight?
date: 2024-09-17 11:48:00 +0500
categories: [Blogs, Software Engineering]
tags: [softwareengineering, objectorientedprogramming, systemdesign, scalability, maintainability, polymorphism, abstraction]
---

![Polymorphism: Attribution: https://pixabay.com/users/mikadago-328028/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=543390](/assets/Polymorphism/polymorphism.png)

Yes. Today, rather than revisiting the traditional *Animal->Cat* and *Animal->Dog* example, we're going to look at a real-life example of polymorphism that all Python developers probably interact with on a daily basis. But first, let's clarify what polymorphism actually is.

According to ChatGPT, 

>Polymorphism is a fundamental concept in object-oriented programming that allows objects of different types to be treated as if they are of the same type. It enables a single interface to be used for different underlying forms (data types).

In simple terms, polymorphism allows for loosely coupled code by enabling the use of a base class (or interface) throughout the codebase while extending its functionality.
I wouldn't say the traditional Cat/Dog example is bad, but if you're like me, you probably need something that resonates with your daily coding experience to truly grasp the concept. So, here goes...

We've all encountered Exceptions in Python, right? Even the most experienced Python developer is bound to make a mistake and face an Exception. However, different errors trigger different types of exceptions, like the `FileNotFoundError` or `OSError`. Now if we just want these exceptions to terminate the execution of our code and tell us what's wrong, raising a specific kind of exception is fine. But what if we want to catch those exceptions and handle errors gracefully? Do we keep adding except blocks for each type of error? That would probably cost you development years. This is where polymorphism comes in!

All exceptions are built upon a base class called Exception, in Python. A custom definition of an exception in Python would look something like this:

```python
class JustARandomException(Exception):
    """

	Just a random exception to show what custom exceptions look like

	"""
```

Essentially, our exception is derived from the built-in `Exception` class in Python. Now you might be wondering how this helps decouple our code. Don't worry, we're here to solve that mystery. Imagine a function XYZ performing a series of operations. In that series of operations, there's a possibility of encountering countless types of exceptions. However, since it's production code, you don't want the application to crash. You just want to show the end-user a graceful error and allow the main thread to keep running. Here's what Python allows you to do:

```python
def func():
	try:

		do_something()

		do_something_else()

	except Exception as e:

		print(f"An unexpected error occurred: {e}")
```

Now, you don't have to add multiple `except` blocks to handle the infinite possibilities for the types of errors, but you'll get to know the type of error in the variable "e". What this means is all exceptions are essentially of type `Exception`. Because of polymorphism, all you need to do is add a generic Exception handler while throwing countless types of custom exceptions throughout your code. 

Just to be clear, use cases vary, and this type of exception handling might not be suitable for every situation. The point of this post was just to explain polymorphism with a concept you're already familiar with. Here's a few ways you might want to handle exceptions:

```python
try:
	do_something()

except Exception as e:

	if isinstance(e, ValueError):

		print("Some line that tells the user how they could avoid this error")

	else:

		print(f"An unexpected error occurred: {e}")
```

or

```python
try:
	do_something()

except ValueError:

	print("Some line that tells the user how they could avoid this error")

except Exception as e:

	print(f"An unexpected error occurred: {e}")
```

Note that in a production environment, showing the exact error message thrown by an exception is considered a bad security practice, so something like this might be more suited:

```python
try:
	do_something()

except Exception as e:

	print(f"An unexpected error occurred. Please contact our support team for further assistance")
```

Anyway, this discussion might be a slight deviation from our main topic. In simple words, the except block only accepts the Exception type as an argument, which allows the developer to add exception handling throughout the code, or even across multiple services, in a more flexible manner.