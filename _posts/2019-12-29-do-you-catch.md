---
layout: post
title: Do You Catch?
---
# Do You Throw?


*Why I chose to use exceptions instead of error codes in my C++ project.*


![img](https://asseo.tech/wp-content/uploads/2019/12/cpp_exceptions_main.png)

# Introduction

As a C programmer starting a new Modern C++ design from scratch, with no predefined and dependent error policy handling, I struggled about the **Error Handling Policy** – should I be using the known and familiar **Error Codes**, or the unfamiliar (to me) – **Exceptions**.

Error codes are solid and functional – it is obvious to C programmers.
If something has been there for a while it must mean it is the correct way, right? No.
Does it mean it is wrong? Neither.
I just think that it is healthy and encouraged to revisit the already accepted norm. That’s how we grow and develop.



### Error Handling Policy

Error handling policy is all about how to report and handle encountered errors in your software.
Errors will appear. there is no doubt.
In the software engineering field we don’t have the privilege to assume things. We have to analyze and handle each possible scenario. You can close your eyes, and leave the door open to the the mixture of ‘luck’ and encountered ‘bugs’ to cook the result for you.
In some “nice” cases it could lead to unknown error with no impact.
In worse cases it could lead to adding more money to a transaction, increasing the air pressure on an airplane cabin, letting in a malware or crashing an aircraft far far away.

When we face the errors and handle them – we are in charge.
We decide what should be done **in case** certain things gone south.

Errors may appear from the physical the the logical.
Physical could be an electromagnetic field affecting our electric wire transfers, causing the transferred frames/packets to become malformed (thank you [CRC](https://en.wikipedia.org/wiki/Cyclic_redundancy_check) for taking care of it).
Logical could be an error in a framework/library you are using, our on code, the operating system, etc.
All of these should be handled. don’t keep your eyes closed. don’t be naive.



# Error Codes

**Error codes** way of error handling is the use of error codes / return values to let the caller function know if the operation was successful or not. and if not – based on the error code “explain” why it has failed.

`Example 1 – Error codes`

{% highlight c++ %} 
`// config_api.h`



`// Prepare configuration.`

`// @return true on success, false if not`

`bool PrepareConfig();`



`// Install configuration.`

`// @return true on success, false if not`

`bool InstallConfig();`
{% endhighlight %} 


{% highlight c++ %} 
`#include "config_api.h"`



`bool InstallDeviceConfig() {`

  `if(!PrepareConfig()) {`

​    `// handle the error..` 

​    `return false;`

  `}`

  

  `if(!InstallConfig()) {`

​    `// handle the error..`

​    `// should I revert the work done by PrepareConfig now?..`

​    `return false;`

  `}`

`}`
{% endhighlight %} 


Other example would be the Linux famous [errno](http://man7.org/linux/man-pages/man3/errno.3.html).





# Exceptions

**Exceptions** way of error handling means that a callee function will attempt to perform it’s job. If an unrecoverable error is encountered – it will throw. The failure information is inside the thrown exception.



`Example 2 – Exceptions`

`// config_api.h`



`// Prepare configuration.`

`// @exception ConfigException - in case of preparation failure`

`void PrepareConfig();`

 

`// Install configuration.`

`// @exception ConfigException - in case of installation failure`

`void InstallConfig();`



`#include "config_api.h"` 

bool InstallDeviceConfig() {

  `try {`

​    `PrepareConfig();`

​    `InstallConfig();`

  `catch(const FileException&amp; ex) {`

​    `// handle specific error`

  `}`

  `catch(const std::exception&amp; ex ) {`

​    `// handle other errors`

  `}`

`}`



# The Bigger Picture

That’s all the differences? should I just use try and catch and wholla – I’m an exceptions user?
When I was looking for knowledge sources about exceptions – I usually encountered such small examples. Small try and catches. I really didn’t understand why would I bother using it.

But after pushing it further and keep reading books, articles, viewing lectures – the bell rang. I saw the matrix. it all made sense to me.

Forget about the try-catch. **Exceptions are about transfer of control**.

In my opinion Exceptions are more of a design choice than a technique.
Transfer of control? Yes -from the point of failure to the entity which has the knowledge to decide from what should be done about it your application perspective. I will explain.

A software is generally built from layers.
At the bottom – we have small functions which perform a single task – open a file, calculate something, allocating memory, etc.
Each upper layer, uses the functionalities and utilities of other functions and modules to build their software logic.
from the simplicity of opening a file. writing to a file and getting the time – I’m able to have the functionality of writing the current time into a file.

Now, what happens if open file has failed. Let’s even go lower – what happen if the open file in glib.c library function has failed because the Linux system call [open](http://man7.org/linux/man-pages/man2/open.2.html) has failed.

Now, as being an amazing and a single purpose function – open knows how to open a file. If it can’t – it fails with the reason why.
The users of open function can be a variety of applications:

- Logging facility
- Database
- Scientific software

The open is generic and can be re-used by all of the above and more. Per this design -if it fails, it doesn’t have the knowledge of the application to decide what should be done about this failure.
Should it terminate? should it retry? should it ignore the failure?
Therefore – it reports the error.

Afterwards, the layer above, in our case glib.c open function, checks the return value of the open system call, and in case it has failed – it will most certainly fail as well.
The next layer, which could be a library generic function like OpenLog – wouldn’t know what to do from our application perspective so it passes failure code as well.

Eventually, if every function really checked the error code (which can be easily ignored) – we reach to a function which can actually make a decision from the application logic perspective.
If it couldn’t load a configuration file for example – maybe it needs to terminate, or maybe it will load with the default settings? maybe it will try to fetch the file from remote location? the application logic knows.

Did you see what happened? we had an **error code domino effect**.
One “leaf” (bottom) function has failed – and each function above was failed like a domino piece, and reported to it’s caller, this we reached to a function with enough application logic information who can actually do something.

This is a real life example – where many functions exist between the error and the handler.

With exceptions – the idea is to **throw** when we encounter an unrecoverable error in a function, and to **catch** on a function which can decide what do to about it.

Maybe we would catch on the same function, and try an alternative way (I couldn’t read the config? let’s create it, maybe it’s the first time the software is loading up).
Maybe we would catch it far away.

Remember – from the application perspective the failure is not about failing to open a file.
The application needed to load it’s configuration. that’s the application logic. and if it failed, it’s the only one who knows what happens next.

# Pros and Cons

Reasons I chose exceptions:

1. Separating the good path from the bad path
2. Application logic versus leaf functions
3. Cannot be ignored
4. STL and Boost has chosen for us

How to design the exceptions:

1. Where to catch?
2. Pitfalls

Application logic vs leaf / end functions5 logs against 1STL and Boost made this decision already
Im a positive guy.
It’s about transfer of control to the function in charge.
sed to replace return with throw

**Don’t Do:**
– Exceptions require the code to be exception safe. you cannot just introduce exceptions to an exiting code without making it exception safe (check – RAII, Stack unwinding, on which context throws are NOT allowed)

