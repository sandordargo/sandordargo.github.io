Do not define a variadic function
Variadic functions can accept a variable number of parameters. The probably best known
example is printf(). You have the possibility to define this kind of functions by yourself but
this is a possible security risk. The usage of variadic functions is not type safe and the wrong
input parameters can cause a program termination with an undefined behavior. This
undefined behavior can be exploited to a security problem. If you have the possibiltiy to use
a compiler that supports C++11, you can use variadic templates instead.
It is technically possible to make typesafe C-style variadic functions with some compilers

https://dwheeler.com/essays/heartbleed.html

