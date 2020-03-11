# What is undefined behaviour

Renders the entire program meaningless if certain rules of the language are violated.

Undefined behaviour occurs when a program does something the result of which is not specified by the standard.

Implementation-defined behaviour is an action by a program the result of which is not defined by the standard, but which the implementation is required to document. 

Language like Java traps error as soon as they are found but language like C and C++ in few cases keep on executing the code in a silent but faulty manner which may result in unpredictable results. The program can crash with any type of error messages, or it can unknowingly corrupt the data which is a grave issue to deal with.

Code can be:
* ill formed: sytax errors, or diagnosable semantic ones
* ill formed no diagnostic required: like One Definition Rule?
* implementation-defined behavior
* unspecified behavior (order of evalutaion)
* undefined behavior

there are no restrictions on the behavior of the program. Examples of undefined behavior are memory accesses outside of array bounds, signed integer overflow, null pointer dereference, more than one modifications of the same scalar in an expression without any intermediate sequence point (until C++11)that are unsequenced (since C++11), access to an object through a pointer of a different type, etc. Compilers are not required to diagnose undefined behavior (although many simple situations are diagnosed), and the compiled program is not required to do anything meaningful.

# Risks and Disadvantages of Undefined Behavior
The programmers sometimes rely on a particular implementation (or compiler) of undefined behavior which may cause problems when compiler is changed/upgraded. For example last program produces 72 as output in most of the compilers, but implementing a software based on this assumption is not a good idea.
Undefined behaviors may also cause security vulnerabilities, especially due the cases when array out of bound is not checked (causes buffer overflow attack).

# Advantages of Undefined Behavior
C and C++ have undefined behaviors because it allows compilers to avoid lots of checks. Suppose a set of code with greater performing array need not keep a look at the bounds, which avoid the needs for complex optimization pass to check such conditions outside loops. The tightly bound loops an speed up the program from thirty to fifty percent when it gains an advantage of the undefined nature of signed overflow, which is generally offered by the C compiler.
We also have another advantage of this as it allows us to store a variable’s value in a processor register and manipulate it over the time that is larger than the variable in the source code. It also helps in wrap-around then compile-time checks which would not be possible without the greater knowledge of the undefined behavior in C/C++ compiler.

# Types of undefined behaviour

## Signed overflow
## Access out of bounds
## Uninitialized scalar
## Invalid scalar
## Null pointer dereference
## Access to pointer passed to realloc
## Infinite loop without side-effects

<!-- stacko https://stackoverflow.com/questions/367633/what-are-all-the-common-undefined-behaviours-that-a-c-programmer-should-know-a -->
## Dereferencing a pointer returned by a "new" allocation of size zero
## Using pointers to objects whose lifetime has ended (for instance, stack allocated objects or deleted objects)
## Converting pointers to objects of incompatible types
## Left-shifting values by a negative amount (right shifts by negative amounts are implementation defined)
## Shifting values by an amount greater than or equal to the number of bits in the number (e.g. int64_t i = 1; i <<= 72 is undefined)
## Casting a numeric value into a value that can't be represented by the target type (either directly or via static_cast)
## Using an automatic variable before it has been definitely assigned (e.g., int i; i++; cout << i;)
## Not returning a value from a value-returning function
## Recursively re-entering a function during the initialization of its static objects
## Exceeding implementation limits (number of nested blocks, number of functions in a program, available stack space .


# Examples
https://www.geeksforgeeks.org/sequence-points-in-c-set-1/ 
1
2
3
https://www.geeksforgeeks.org/delete-this-in-c/
https://www.geeksforgeeks.org/accessing-array-bounds-ccpp/
https://www.geeksforgeeks.org/execution-printf-operators/ 
https://www.geeksforgeeks.org/virtual-destructor/

READ https://blog.regehr.org/archives/213
https://www.nayuki.io/page/undefined-behavior-in-c-and-cplusplus-programs


# How to exploit undefined behaviour


# References
https://en.cppreference.com/w/cpp/language/ub
https://en.cppreference.com/w/cpp/language/definition
https://www.geeksforgeeks.org/undefined-behavior-c-cpp/



# C++OnSea


## Define Undefined Behaviour

## Show Undefined Behaviour in running code

When you access an element outside a container of the STL, the result is not so promising. You effect may be an error or undefined behaviour. Undefined behaviour means all bets are open

https://www.modernescpp.com/index.php/c-core-guidelines-avoid-bound-errors


## List undefined behaviours in the STL (containers at vs operator[], general UBs in algorithms through invalid parameters, transform, copy_n, swap, is_permutation)

## Rational behind UBs: shorter API, simpler implementation, performance optimization

## Why undefined behaviour is allowed in the STL

STL almost never checks for logical errors => no exceptions due to logical pobs
only `at()`! (vectors and deques)

late standardization of exceptions
performance matters

no perf leak + no container invariants (???) violation
*****
CONTNIUE HERE
******

https://books.google.fr/books?id=n9VEG2Gp5pkC&pg=PA139&lpg=PA139&dq=stl+undefined+behaviour&source=bl&ots=Rgm2rj9aQS&sig=ACfU3U36MSxzbwEvaRUxy_HfotBd3g4ePw&hl=en&sa=X&ved=2ahUKEwj90ce66LfnAhXBz4UKHXfzB5MQ6AEwA3oECAYQAQ#v=onepage&q=stl%20undefined%20behaviour&f=false


----
When undefined behaviour is allowed, it's usually for reasons of efficiency.

If the standard specified what has to happen when you access an array out of bounds, it would force the implementation to check whether the index is in bounds. Same goes for a vector, which is just a wrapper for a dynamic array.

In other cases the behaviour is allowed to be undefined in order to allow freedom in the implementation. But that, too, is really about efficiency (as some possible implementation strategies could be more efficient on some machines than on others, and C++ leaves it up to the implementer to pick the most efficient strategy, if they so desire.)
https://stackoverflow.com/questions/22059258/why-is-undefined-behavior-allowed-in-the-stl


According to Herb Sutter one marked reason is efficiency. He states that the standard does not impose any requirements on operator[]'s exception specification or whether or not it requires bound checking. This is up to the implementation.
http://www.gotw.ca/gotw/074.htm

### Performance for vector
https://stackoverflow.com/questions/3269809/why-is-stdvectoroperator-5-to-10-times-faster-than-stdvectorat
https://stackoverflow.com/questions/9376049/vectorat-vs-vectoroperator

### check what happens for vector::operator[]() with different implementations
Always checking bounds would cause a (possibly slight) performance overhead on all programs, even ones that never violate bounds. The spirit of C++ includes the dictum that, by and large, you shouldn't have to pay for what you don't use, and so bounds checking isn't required for operator[]()

## Go through the UBs one by one (which ones will depend on the length on the slot given)

## How to protect ourselves through learning, documentation, assertions, custom checks and constructs

## Conclusion