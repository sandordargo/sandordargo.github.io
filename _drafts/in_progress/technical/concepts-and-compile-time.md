concepts-and-compile-time.md

slower than without

not slower than with templates

------
requries and trailing requries closed

Hi Andrei,

Thanks for your question during the concepts of concepts presentation.

So what's the difference between requries and trailing requries clause?

You cannot use requires clause when you have a class template and you want to provide overloads based on concepts. You can only use the trailing requires clause. (slide 30 / 00:25).

Otherwise, think about the differences between normal function declarations and function declarations with trailing return types. In the trailing version the enclosing cl