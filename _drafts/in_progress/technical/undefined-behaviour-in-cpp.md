https://www.copperspice.com/pdf/Undefined-Behavior-CppCon-2018.pdf

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

### In general operations are not safe. The caller must ensure that the parameters of the operations meet the requirements. Violating the requirements is UB and usually the STL doesn't throw exceptions by itself.
	
### Performance for vector
https://stackoverflow.com/questions/3269809/why-is-stdvectoroperator-5-to-10-times-faster-than-stdvectorat
https://stackoverflow.com/questions/9376049/vectorat-vs-vectoroperator

### check what happens for vector::operator[]() with different implementations
Always checking bounds would cause a (possibly slight) performance overhead on all programs, even ones that never violate bounds. The spirit of C++ includes the dictum that, by and large, you shouldn't have to pay for what you don't use, and so bounds checking isn't required for operator[]()

## Go through the UBs one by one (which ones will depend on the length on the slot given)

### vector at is not! [] is, list like front() and back[] on empty. some runtime error

### Calling erase(), invalidates pos, soyou cannot call ++ on it, undefined behaviour
Screenshot from 2020-02-06 13-19-42.png

### coll.erase(coll.end()) UB
### pop_front/pop_back on empty
### list:splice this and source must be different and pos must be in a valid place / sourcepos too
### Note that before_begin() and cbefore_begin() do not represent a valid position of a Forward List. Therefore, dereferencing these positions results in undefined behavior.
### On a Forward List  appending element after end is undefined behavior
### push_frong with (&&) • The second form, which is available since C++11, moves value to the container, so the state of value is undefined afterward.
### Passing end() or cend() of a container as pos results in undefined behavior.
###size_type container::bucket (const key_type key) const

• Returns the index of the bucket in which elements with a key equivalent to key would be found, if any such element existed.

• The return value is in the range [0,bucket_count()).

• The return value is undefined if bucket_count() is zero.
### size_type container::bucket_size (size_type bucketIdx) const

• Returns the number of elements in the bucket with index bucketIdx.

• If bucketIdx is not a valid index in the range [0,bucket_count()), the effect is undefined.

### default constructor • For arrays, the operation is implicitly defined and creates a nonempty container where the elements might have undefined value

### pos < coll.end()-1

If the collection was empty, coll.end()-1 would be the position before coll.begin(). The comparison might still work, but, strictly speaking, moving an iterator to before the beginning results in undefined behavior. Similarly, the expression pos += 2 might result in undefined behavior if it moves the iterator beyond the end() of the collection. Therefore, changing the final loop to the following is very dangerous because it results in undefined behavior if the collection contains an odd number of elements (Figure 9.2):

### Adapters not1() and not2()
The adapters not1() and not2() can be considered as almost deprecated.5 The only way to use them is to negate the meaning of predefined function objects. For example:

5 In fact, they were close to being deprecated with C++11, see [N3198:DeprAdapt]

std::sort (coll.begin(), coll.end(),
           std::not2(std::less<int>()));

This looks more convenient than:

Click here to view code image

std::sort (coll.begin(), coll.end(),
           std::bind(std::logical_not<bool>(),
                     std::bind(std::less<int>(),_1,_2)));

However, there is no real real-world scenario for not1() and not2() because you can simply use another predefined function object here:

std::sort (coll.begin(), coll.end(),
           std::greater_equal<int>());

More important, note that calling not2() with less<> is wrong anyway. You probably meant to change the sorting from ascending to descending. But the negation of < is >=, not >. In fact, greater_equal<> even leads to undefined behavior because sort() requires a strict weak ordering, which < provides, but >= does not provide because it violates the requirement to be antisymmetric (see Section 7.7, page 314). Thus, you either pas
san

###  However, incrementing the end() of a collection results in undefined behavior
### Without using inserters, copy() would overwrite the empty collection coll2, resulting in undefined behavior
### Rotating Elements - The caller must ensure that newBeg is a valid position in the range [beg,end); otherwise, the call results in undefined behavior.
### According to the standard, calling SORTED-RANGE ALGORITHMS for sequences that are not sorted on entry results in undefined behavior. 
### std::priority_queue top() and pop(), back()

### string Note that for operator [], the number of characters is a valid index, returning a character representing the end of the string. This end-of-string character is initialized by the default constructor of the character type ('\0' for class string):  Before C++11, for the nonconstant version of operator [], the current number of characters was an invalid index. Using it did result in undefined behavior.

### s[s.length()]                   // yields '\0' (undefined behavior before C++11)

### To enable you to modify a character of a string, the nonconstant versions of [], at(), front(), and back() return a character reference. Note that this reference becomes invalid on reallocation:

### string(const char* cstr): • Note that passing a null pointer (nullptr or NULL) results in undefined behavior, operatir=() or assign() as well

### • Note that the character at the reverse end is not defined. Thus, *s.rend() and*s.crend() result in undefined behavior.

### streams Since C++11, some guarantees regarding concurrency are given for these stream objects: When synchronized with the standard streams of C, using them in multiple parallel threads does not cause undefined behavior. Thus, you can write from or read into multiple threads. Note, however, that this might result in interleaved characters, or the thread that gets a character read is undefined. For any other stream object or if these objects are not synchronized with C streams, concurrent reads or writes result in undefined behavior.

### ostream& ostream::write (const char* str, streamsize count) - The caller must ensure that str contains at least count characters; otherwise, the behavior is undefined.

### file.seekg (-10, std::ios::end); - In all cases, care must be taken to position only within a file. If a position ends up before the beginning of a file or beyond the end, the behavior is undefined.

### std::istream in (out.rdbuf()); -  If it is opened only for writing, reading from the stream will result in undefined behavior 

### std::use_facet<std::numpunct<char>>(loc).truename()
  - Note that use_facet() returns a reference to an object that is valid only as long as the locale object exists. Thus, the following statements result in undefined behavior because fac is no longer valid after the first expression:
  const numpunct<char>& fac = use_facet<numpunct<char>>(locale("de"));
cout << "true in German: " << fac.truename() << endl;  // ERROR


### Unless the policy class defines a virtual destructor, applying delete to a pointer tothe policy class has undefined behavior, (Alexandrescu)
### • We run into alignment issues. Imagine you build an allocator for 5-byte blocks. In this case, casting a pointer that points to such a 5-byte block to unsigned int engenders undefined behavior. This is the big problem
### To make a long story short, once you use ellipses, you’re left in a world with no type safety, no object semantics (using full-fledged objects with ellipses engenders undefined behavior), and no support for reference types. Even the number of arguments is not accessible to the called function. Indeed, where there are ellipses, there’s not much C++ left.

### Therefore, Log is destroyed before Keyboard. If for some reason Keyboard fails to shut down and tries to report an error to Log, Log::Instance unwittingly returns a reference to the “shell” of a destroyed Log object. The program steps into the shady realm of undefined behavior. This is the dead-reference problem.

### Checking before dereference is important because dereferencing the null pointer engenders undefined behavior. For many applications, undefined behavior is not acceptable, so checking the pointer for validity before dereference is the way to go

### allocation failure during swap is defined beahacour

### calling swap on container instances whose allocatiors do not compare equally is undefined behaviour (can be a different allocator) -> memory corruptrion and segfault

## How to protect ourselves through learning, documentation, assertions, custom checks and constructs



## Conclusion