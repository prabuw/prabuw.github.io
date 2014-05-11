---
layout: post
title: Answers to .NET Developer Interview Questions
categories:
- Interview Questions
tags:
- Interview Questions
- Interview Questions Answers
- Technical Interview
status: publish
type: post
published: true
meta:
  _publicize_pending: '1'
author: 
---
Last year I posted a list of [questions](/posts/senior-developer-interview-questions) that for developer position in .NET. Some people that came across the article
asked for the answers. I will try to answer them as best I know, although it is always best to do some research on your
own as well.

Since some of the questions are quite broad, which are trying to gauge the depth of my knowledge, and for these I will
simply place links to **Stackoverflow** answers or articles that I think are good if not great.

Also, if the answer is wrong, or if the links are broken, please do not hesitate to tell me, if not for me to correct my
understanding, but more importantly so I don't perpetuate misinformation.

####What are things you should be mindful of when developing for an application that is going to be hosted on multiple servers with load balancing in place?####

####Can you describe how to implement cancellation in the context of a multi-threaded application?####
####Can you explain what the **Lock** keyword is used for?####

The lock keyword is used in multithreaded applications to ensure that the current thread executes a block of code to
completion without interruption by other threads.&nbsp; This means that if done correctly, we can reliably say that a
single thread is executing the code within the lock.

A wonderful explanation of the use of the lock keyword can be found on this [stackoverflow answer](http://stackoverflow.com/questions/9621438/confusion-about-the-lock-statement-in-c-sharp")
by Eric Lippert, a former member of the C# language design team at Microsoft. You can find the MSDN article for the
lock keyword [here](http://msdn.microsoft.com/en-us/library/c5kehkcz.aspx).

####Can you explain what semaphores and mutexes are?####

Semaphores and mutexes are controls in concurrent programming. In concurrent programming, multiple threads may require
access to a shared resource.&nbsp; A common analogy to explain this is as follows:

  - A mutex is a like a key to a bathroom. One person can have the key and occupy the bathroom at a time. When the
    person is finished with the bathroom, the key is given to the next person in the queue.
  - A semaphore is a count that describes the number of free identical bathroom keys. Initially the count is equal to
    the number of free bathrooms (e.g. 4). As each key is acquired and bathroom is used, the count is decremented.
    Once the use of the key is freed, the count is incremented.

A more complete understanding of these concepts can be found [here](http://msdn.microsoft.com/en-us/library/ms228964\(v=vs.110\).aspx)

####What is dynamic polymorphism?####

Dynamic polymorphism is another term for method overriding. This is evaluated at run-time, as oppose to compile time.
Static polymorphism, also known as method overloading is done at compile time. To get a complete understanding of this
and polymorphism in general in a .NET context, visit this [MSDN article](a href="http://msdn.microsoft.com/en-us/library/ms173152.aspx).

####How does SSL work?####

In my attempts to grasp a better understanding of this topic myself, I wrote this [explanation](/posts/how-does-ssl-work/) myself.
While it might be daunting at first, it is quite straightforward after reading it several times
(and writing my own understanding of it as well).

####Can you explain how SQL injection works and how would you guard against it?####

SQL Injection is a potential vulnerability that can be found in any application with a database backend. It is the
number one vulnerability on [OWASP](https://www.owasp.org/index.php/Main_Page "Open Web Application Security Project").
The following example of building inline SQL queries will generally expose this problem.

``` csharp
var query = "SELECT * FROM Users WHERE ID = " + Request.QueryString["ID"];
```

A user can manipulate the query string to modify the SQL query and thereby retrieve sensitive data or perform malicious
action.

SQL injection can generally be prevented by parameterising inputs to the queries. This can be achieved by using a
reputable ORM such as Entity Framework or NHibernate, or building parameterised queries with ADO.NET.

A good explanation of SQL injection by Troy Hunt can be read [here](http://www.troyhunt.com/2013/07/everything-you-wanted-to-know-about-sql.html)

####What is cross site scripting and how do you prevent it?####

Cross-site scripting (XSS) is a potential vulnerability found typically in web applications, where a piece of
client-side script is injected into a web page that is viewed by other users. Once other users view this page, the
injected client-side script is then executed.

A simple example of this is:

+ A user enters the following script tag such as ` <script>alert('Hello')</script> ` in to a form on a message board and submits
  the form.
+ The form data (i.e. the message) is stored into the database.
+ The next time any user navigates to the page where this message is displayed, the script is executed, which in this
case pops up a harmless but annoying alert. More malicious scripts can be written to hijack a user's session,
re-direct users to other pages, and even by-passing the [same-origin-policy](http://en.wikipedia.org/wiki/Same-origin_policy "same-origin policy").

Preventing this can be done by encoding the input from forms and output as html on to the page. This is done by
replacing potentially unsafe characters such as the greater than character (>) with HTML encoded equivalent, which in
this case is *\&\#62;*. ASP.NET has syntax and methods to achieve this built in.

Here are some good reading on the topic:

+ A great [video](http://channel9.msdn.com/Events/MIX/MIX10/FT05) by Scott Hanselman and Phil Haack about
  vulnerabilities and methods to prevent them using ASP.Net MVC.
+ A lengthy but sound write up can be found [here](http://www.troyhunt.com/2010/05/owasp-top-10-for-net-developers-part-2.html")
  by Troy Hunt.

####What is the difference between a.Equals(b) vs. a == b ?

The short hand answer to this question is:

+ The == operator is there to check the identity equality. I.e. Is object a the same as object b.
+ The .Equals() method is checking the value equality. I.e. Is object a and object b holding the same values.

There is a notable exception which is for strings and predefined value types (e.g. int, float etc.), the == operator
will check for value equality and not identity equality, which is the same as performing .Equals().

Please read [this](http://msdn.microsoft.com/en-us/library/ms173147.aspx) excellent answer to this question on MSDN.

####Can you talk a bit about object design patterns and what you know about them?

####What is cross site forgery and how do you guard against it?

Cross site forgery, also know as CSRF or XSRF, is a vulnerability found on web applications. It involves a malicious
site sending a request to a vulnerable site, where the user is already logged in. An common example of an attack is
as follows:

+ First of all, this attack requires the user being authenticated on the site being attacked. For this example, let us
  make that www.mybank.com
+ A user is directed to render an HTML that contains an iFrame or an image with a source  such as:
    + `<a href="http://mybank.com/withdraw?fromaccountid=100&amp;amount=1000000&amp;toaccounit=200">http://mybank.com/withdraw?fromaccountid=100&amp;amount=1000000&amp;toaccounit=200</a>`.
+ The user might have been duped into going to this site via an email, which is part of the scam.
+ As soon as the iFrame or image renders, the HTTP request is made and a transfer of funds is done, unknown to the user.

This problem can be avoided by using an anti-forgery token, which is built into the .NET libraries. A good explanation
of how to implement this can be found [here](http://www.asp.net/web-api/overview/security/preventing-cross-site-request-forgery-(csrf)-attacks).
Also have a read of Steve Sanderson's article on it [here](http://blog.stevensanderson.com/2008/09/01/prevent-cross-site-request-forgery-csrf-using-aspnet-mvcs-antiforgerytoken-helper/),
as it explains the internal workings of ASP.NET MVC's anti-forgery token, and the limitations of it.

####Explain what the Big O notation is.

Big O notation is a computer science term to describe the relative complexity of an algorithm. It describes the
algorithm in regards of time and space, using the quantity of items passed as a descriptor.<br><br>An excellent
stackoverflow answer of this question can be found [here](http://stackoverflow.com/questions/487258/plain-english-explanation-of-big-o).

####What is a binary tree?

####How does the garbage collector work?

####What is normalisation and de-normalisation?

####Can you speak a bit about database indexes and what you know off them.

####How do you store passwords in a database?

####How does hashing and salting work?

####If you had to how do you test private methods?

Personally I prefer not to test private methods, as they are usually an implementation detail that should be hidden to
the users of the class. If there is a need to test one, it might be worth refactoring part or all of it into a
public method. Also, we can convert the private method into an [object method](http://www.refactoring.com/catalog/replaceMethodWithMethodObject.html)
to allow easier testing.

However, if the method must remain private, we can gain access to test it using the [InternalsVisibileToAttribute](http://msdn.microsoft.com/en-us/library/system.runtime.compilerservices.internalsvisibletoattribute\(v=vs.110\).aspx "InternalsVisibileToAttribute")
class.

####What is the IDisposable interface, and what does it achieve for a class implementing it?

####What is optimistic concurrency?

####What are dynamic proxies?
