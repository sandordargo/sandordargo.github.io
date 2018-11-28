Reading through Scott Meyer's Efective Modern C++ helped me discover a lot of features of modern C++, including right value references, the trailing return type declaration and lamdba expressions. Let's talk about those lambdas right now.

You might think, come on, this is old stuff, every serious developer should know about lamdba experssions. You might be right, yet, it's not the case. Recently I made a brown bag session on lambdas and out of about 15 developers, two of us have already used lambdas in C++ and two others in Java. So the need is out there.

What are lamdba expressions?

Lamdba expressions are anonymous functions. They are small snippets of code that provide a better readability in most cases if they are not hidden into an enclosing class. By th way, in C++, those enclosing classes would be called functors or function objects. We are going to cover them in a minute.

So we can say, that lambda expressions are here for us to replace functors and to make the code more expressive. Through their ease of usage and extreme expressivity, they boost the usage of the Standard Template Library.

At this point, I have to make a confession. I'm used to be a 