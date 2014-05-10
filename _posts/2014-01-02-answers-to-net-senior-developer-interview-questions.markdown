---
layout: post
title: Answers to .NET Senior Developer Interview Questions
categories:
- Interview Questions
tags:
- Interview Questions
- Interview Questions Answers
- Senior Developer Interview
- Technical Interview
status: publish
type: post
published: true
meta:
  _publicize_pending: '1'
author: 
---
<p align="justify">Last year I posted a list of questions that I got asked for a senior developer position in .NET. Some people that came across the article asked for the answers. I will try to answer them as best I know, although it is always best to do some research on your own as well.</p>
<p align="justify">Since some of the questions are quite broad, and the interviewer was trying to gauge the depth of my knowledge, and for these I will simply place links to stackoverflow answers or articles that I think are good if not great.</p>
<p align="justify">Also, if the answer is wrong, or if the links are broken, please do not hesitate to tell me, if not for me to correct my understanding, but more importantly so I don't perpetuate misinformation.</p>
<ol>
<li>
<div align="justify">What are things you should be mindful of when developing for an application that is going to be hosted on multiple servers with load balancing in place? </div>
<li>
<div align="justify">Can you describe how to implement cancellation in the context of a multi-threaded application? </div>
<li>
<div align="justify">Can you explain what the <em>Lock</em> keyword is used for? </div>
<div align="justify"><font color="#000000"><font color="#000080">The lock keyword is used in multithreaded applications to ensure that the current thread executes a block of code to completion without interruption by other threads.&nbsp; This means that if done correctly, we can reliably say that a single thread is executing the code within the lock. <br><br>A wonderful explanation of the use of the lock keyword can be found on this <a href="http://stackoverflow.com/questions/9621438/confusion-about-the-lock-statement-in-c-sharp" target="_blank">stackoverflow answer</a> by Eric Lippert, a former member of the C# language design team at Microsoft. You can find the MSDN article for the lock keyword <a href="http://msdn.microsoft.com/en-us/library/c5kehkcz.aspx" target="_blank">here</a>.</font><br></div>
<li>
<div align="justify">Can you explain what semaphores and mutexes are? </div>
<div align="justify"><font color="#000080">Semaphores and mutexes are controls in concurrent programming. In concurrent programming, multiple threads may require access to a shared resource.&nbsp; A common analogy to explain this is as follows: <br></font>
<ul>
<li><font color="#000080">A mutex is a like a key to a bathroom. One person can have the key and occupy the bathroom at a time. When the person is finished with the bathroom, the key is given to the next person in the queue. </font>
<li><font color="#000080">A semaphore is a count that describes the number of free identical bathroom keys. Initially the count is equal to the number of free bathrooms (e.g. 4). As each key is acquired and bathroom is used, the count is decremented. Once the use of the key is freed, the count is incremented. </font></li>
</ul>
<p><br><font color="#000080">A more complete understanding of these concepts can be found </font><a href="http://msdn.microsoft.com/en-us/library/ms228964(v=vs.110).aspx" target="_blank">here</a><font color="#ff0000"><font color="#000080">.</font> </font><br></div>
<li>
<div align="justify">What is dynamic polymorphism? </div>
<div align="justify"><font color="#000000"><font color="#000080">Dynamic polymorphism is another term for method overriding. This is evaluated at run-time, as oppose to compile time. Static polymorphism, also known as method overloading is done at compile time. To get a complete understanding of this and polymorphism in general in a .NET context, visit this</font> </font><a href="http://msdn.microsoft.com/en-us/library/ms173152.aspx" target="_blank">MSDN article</a><font color="#000000"><font color="#000080">.</font> </font></div>
<p><br>
<li>
<div align="justify">How does SSL work? </div>
<div align="justify"><font color="#000000"><font color="#000080">In my attempts to grasp a better understanding of this topic myself, I wrote this <a href="http://pwee167.wordpress.com/2013/07/21/how-does-ssl-work/" target="_blank">explanation</a> myself. While it might be daunting at first, it is quite straightforward after reading it several times (and writing my own understanding of it as well).</font></div>
<p><br>
<li>
<div align="justify">Can you explain how SQL injection works and how would you guard against it? </div>
<div align="justify"><font color="#000000"><font color="#000080">SQL Injection is a potential vulnerability that can be found in any application with a database backend. It is the number one vulnerability on <a href="https://www.owasp.org/index.php/Main_Page" target="_blank"><abbr title="Open Web Application Security Project">OWASP</abbr></a>.&nbsp; The following example of building inline SQL queries will generally expose this problem. <br><br><code>var query = "SELECT * FROM Users WHERE ID = "+ Request.QueryString["ID"];</code> </div>
<p><br><br>A user can manipulate the query string to modify the SQL query and thereby retrieve sensitive data or perform malicious action.
<p align="justify"><br>SQL injection can generally be prevented by parameterising inputs to the queries. This can be achieved by using a reputable ORM such as Entity Framework or NHibernate, or building parameterised queries with ADO.NET.<br><br>A good explanation of SQL injection by Troy Hunt can be read <a href="http://www.troyhunt.com/2013/07/everything-you-wanted-to-know-about-sql.html" target="_blank">here</a></font>.<br></p>
<li>
<div align="justify">What is cross site scripting and how do you prevent it? </div>
<div align="justify"><font color="#000000"><font color="#000080">Cross-site scripting (XSS) is a potential vulnerability found typically in web applications, where a piece of client-side script is injected into a web page that is viewed by other users. Once other users view this page, the injected client-side script is then executed.<br><br>A simple example of this is:
<ul>
<li>A user enters the following script tag such as<em> </em><code><script>alert(‘Hello')</script></code> in to a form on a message board and submits the form.
<li>The form data (i.e. the message) is stored into the database.
<li>The next time any user navigates to the page where this message is displayed, the script is executed, which in this case pops up a harmless but annoying alert. More malicious scripts can be written to hijack a user's session , re-direct users to other pages, and even by-passing the <a href="http://en.wikipedia.org/wiki/Same-origin_policy" target="_blank">same-origin policy</a>.</li>
</ul>
<p></font>
<ul></ul>
</div>
<p><font color="#000080"><br>Preventing this can be done by encoding the input from forms and output as html on to the page. This is done by replacing potentially unsafe characters such as the greater than character (>) with HTML encoded equivalent, which in this case is &amp;gt;.&nbsp; ASP.NET has syntax and methods to achieve this built in.<br><br>Here are some good reading on the topic: </p>
<ul>
<li>A great <a href="http://channel9.msdn.com/Events/MIX/MIX10/FT05" target="_blank">video</a> by Scott Hanselman and Phil Haack about vulnerabilities and methods to prevent them using ASP.Net MVC.
<li>A lengthy but sound write up can be found <a href="http://www.troyhunt.com/2010/05/owasp-top-10-for-net-developers-part-2.html" target="_blank">here</a> by Troy Hunt.</font></li>
</ul>
<p><br>
<li>
<div align="justify">What is the difference between a.Equals(b) vs. a == b ?</div>
<div align="justify"><font color="#000000"><font color="#000080">The short hand answer to this question is: <br>
<ul>
<li>The == operator is there to check the identity equality. I.e. Is object a the same as object b.
<li>The .Equals() method is checking the value equality. I.e. Is object a and object b holding the same values. </li>
</ul>
<p><br>There is a notable exception which is for strings and predefined value types (e.g. int, float etc.), the == operator will check for value equality and not identity equality, which is the same as performing .Equals(). <br><br>Please read <a href="http://msdn.microsoft.com/en-us/library/ms173147.aspx" target="_blank">this</a> excellent answer to this question on MSDN. </font><br></div>
<li>
<div align="justify">Can you talk a bit about object design patterns and what you know about them?</div>
<li>
<div align="justify">What is cross site forgery and how do you guard against it?</div>
<div align="justify"><font color="#000080">Cross site forgery, also know as CSRF or XSRF, is a vulnerability found on web applications. It involves a malicious site sending a request to a vulnerable site, where the user is already logged in. An common example of an attack is as follows: <br><br>
<ul>
<li>First of all, this attack requires the user being authenticated on the site being attacked. For this example, let us make that www.mybank.com
<li>A user is directed to render an HTML that contains an iFrame or an image with <code><a href="http://mybank.com/withdraw?fromaccountid=100&amp;amount=1000000&amp;toaccounit=200">http://mybank.com/withdraw?fromaccountid=100&amp;amount=1000000&amp;toaccounit=200</a></code>. The user might have been duped into going to this site via an email, which is part of the scam.
<li>As soon as the iFrame or image renders, the HTTP request is made and a transfer of funds is done, unknown to the user. </li>
</ul>
<p><br><br>This problem can be avoided by using an anti-forgery token, which is built into the .NET libraries. A good explanation of how to implement this can be found <a href="http://www.asp.net/web-api/overview/security/preventing-cross-site-request-forgery-(csrf)-attacks" target="_blank">here</a>.&nbsp; Also have a read of Steve Sanderson's article on it <a href="http://blog.stevensanderson.com/2008/09/01/prevent-cross-site-request-forgery-csrf-using-aspnet-mvcs-antiforgerytoken-helper/" target="_blank">here</a>, as it explains the internal workings of ASP.NET MVC's anti-forgery token, and the limitations of it.</font></div>
<p><br>
<li>
<div align="justify">Explain what the Big O notation is.</div>
<div align="justify"><font color="#000080">Big O notation is a computer science term to describe the relative complexity of an algorithm. It describes the algorithm in regards of time and space, using the quantity of items passed as a descriptor.<br><br>An excellent stackoverflow answer of this question can be found <a href="http://stackoverflow.com/questions/487258/plain-english-explanation-of-big-o" target="_blank">here</a>. </font></div>
<p><br>
<li>
<div align="justify">What is a binary tree?</div>
<li>
<div align="justify">How does the garbage collector work?</div>
<li>
<div align="justify">What is normalisation and de-normalisation?</div>
<li>
<div align="justify">Can you speak a bit about database indexes and what you know off them.</div>
<li>
<div align="justify">How do you store passwords in a database?</div>
<ol>
<li>
<div align="justify">How does hashing and salting work?</div>
</li>
</ol>
<li>
<div align="justify">If you had to how do you test private methods?</div>
<div align="justify"><font color="#000080">Personally I prefer not to test private methods, as they are usually an implementation detail that should be hidden to the users of the class. If there is a need to test one, it might be worth refactoring part or all of it into a public method. Also, we can convert the private method into an <a href="http://www.refactoring.com/catalog/replaceMethodWithMethodObject.html" target="_blank">object method</a> to allow easier testing. <br><br>However, if the method must remain private, we can gain access to test it using the <a href="http://msdn.microsoft.com/en-us/library/system.runtime.compilerservices.internalsvisibletoattribute(v=vs.110).aspx" target="_blank">InternalsVisibileToAttribute</a> class. </font></div>
<p><br>
<li>
<div align="justify">What is the IDisposable interface, and what does it achieve for a class implementing it?</div>
<li>
<div align="justify">What is optimistic concurrency?</div>
<li>
<div align="justify">What are dynamic proxies?</div>
<li>
<div align="justify">What is the difference between a mid level developer and senior developer?</div>
</li>
</ol>
