already socializing at breakfast



Absolutely amazing venue
how did they have my badge in their hand, who knows?

maven/talk/financial trading

many people were already here
sponsorts
St Petersburg...

CoC
I understand 
physical and stereotypes
not entirely the verbal abuse. what is misgendering?
be kind for everyone we meet, that's totally reasonable
even Phil mentions the misgendering, ok, I guess, it's just for those who put their pronouns

online event as well
Hana already tried to do it be in person, but whit happened, flight cancellationm whatever

Hana, Lightning Updates
egy magyar?
chair of reflection study group
Avast researcher
CTRE library

the problem:
snych a dataase on 10 millions of clients

what can you update?
executable
content (db, textures, models)
=> the state of an application is updates

## updating mechanisms
replace everythin
additional/overlay
differential

how do you update from one state of another
update graph!

what is a vocabulary type?

fine as long as you don't define a destructor or add a data member
using super = std::array...
using super::sruper;


> Strong types create strong code - Tony van Eerd

deduce this - constness of this is deduced

crypto is bad
don't use yaml
feels a bit stranfe, but... it gives a color

shared<const T> - can't form a cycle, it won't leak (WHY, that's probably a good topic for an article)
behaves an immutable value, no need for locking
saves memory, we can deal with large objects

i missed an agenda from the presentation, it would have helped to follow where we are and why we are at a given place
hana, what do you use for the graphs in the pres?

change the problem to something simpler, that a very good message

model algorithm into code
strong types makes your code easy to reason about
modern c++ is fun

she also ends with the opening slide

=============
Mateusz Pusz - C++23

minor release, but still lots of things
C++ ISO is a code/paper review committee

(the sound is horrible)

deducing this'
it can deduce a derived type

P0849

std::erase(x. x.front()) not working (check the talk!)

if consteval

if constexpr (is_constant_evaluated) -> always runtime

if(std::is_constant_eval).. not enough it must be constexpr

P2242
non-literal variables in constexpr functions

P1102 down with ()!

the lamdba doesn't need in () if no params


P0798
std::optional
and_then
or_else (either does something or can throw something)
transform

P0323 expected
unlike vairant, it's never empty
API of optional w/o the monadic interface
partial specialization for <void, E>


PIC for example
use it with std::error_code


P0881
stacktrace lib

what the hell is PMR?

P1132 out_ptr, a scalable output pointer abstraction

P1989 range ctor for string_view

P1659
starts_with/ends_with for containers!

P2321
adding zip

P2441
join_with!

std::unreachable!

Arno, error handlinkg


return value - no good for the constructor - use `[[nodiscard]]`
out params are strange 

variant T/error + some functions (std::expected)

exceptions
use &
  throw throws the original exception
  if by value and throws=> terminate

what does it mean to write exception safe code?

in development we have to be paranoid

collect as much info as possible
carry on somehow, don't terminate

exceptions are like goto, lots of new paths to test


===========
J. Coe
composite class design

as much as I love C++, I love to avoid writing code

problem with pointer members (copy...)

avoid writing code
if you have to write code, it's good if the compiler can tell you whether it's correct or not

const propagation = from a const member func calling a func of a member var


what to do when an infrequently used member is very big?

closed-set-polymorphism
vs
open set polymorphism

store pointers to members or collection of..
but with then we went back to the original problem


what about smart pointers?

design our polymorphic_value<B>

indirect_value<T, Copier, Deleter>
so that we can store something big

these are begin standardized

Small Value Optimization not even considered for indirect_value, it would be a lie

for poly, it's a valid choice

old name, cloned_pointer, deep_pointer...


# Day2
optiver still nice and amazing stuff

## Daniela
Contemporary C++ in Action

is C++ really for nerds and dead?

demo application requirements in picture

std::generator? wth? interface in front ot a corotine to iterate over all the particular entities that are created in the coro
libav == ffmpeg! now in module

ez nem egy tul jo prezi
nincs jo kiallasa

valassz eloadot, ne temat

cot=ntemporary should be more than just coros
modules should be covered simpl by "import"


contemporary c++ is 
simple
concise
safe
composable
enoyable

## Dave Winterbottom

C++14

using the library is good, it's sometimes not the standard library, but just "a" library, still it'scode reuse
we should use a toolbox

if soemthing is different it should be importantly different to stand out

how to evaluate a piece of code that you yourself cannot write

clarity in evaluating performance

"optimize for readability"


writeablilty
readabiilty
maintainability
debugability
performance/optimizability


optional for readability
move for performance
auto for maintainability, maybe for debugability? and for optimisability, to get the right type!

"it doesn't make it worse" is not a great answer, but better than nothing and show how ways it's ehlpful

(for example for auto) intentionability is important. you should be able to explain your intentions all the time

override specifier - easier to read, safer
unique_ptr is also like that


lead by example!
which lenses do you use?

lead by example, 

see you in the bar

## bjorn fahhler
what do you mean by cache friendly?

he was not limited by latency, but by CPU?
why???
schedule_timer...

### simplistic model includes
caches is small
fixed size lines
fast when hit
miss slow

### excludes
multiple levels
associativity
threading

question
how do you test that your code is cache friendly?

callgrind! to collect cache hit/miss stats
70% is bad, then what is good?
KCachegrind to visualize

O0 is stupid
O1 removes the stupidity
O3 difficult to understand

timer object is 40 bytes, in a loop of 20k is much bigger than the L1 cache
if you follow a pointer, that's gonna be a cache miss

a vector is a contig store, no chasing pointers

hotspot tool to linux
flame graph

doing more work can be faster than doing less!

confusing branch misses results due to speculative execution

big O still matters, but not all mem accesses are the same

we often see a difference based on the number of items to be stored

heap and prio_queue seems great, but even map was better than expected


following a pointer is a cache miss, unless you have info on the contrary
smaller working data set is better
use as much of a cache entry as you can
sequential mem accesses can be very fast due to prefetching
fewer evicted cache lines means more data in hot cache for the rest of the program
mispredicted barnches can evict cache entries
linear access in contig memory rules for small data sets
measure, measure, measure

[[likely/unlikely]] matters when you do O3 because it might rearrange your brnaches, but it also implies that you know what is {un}likely

perf testing, it's difficult, micro branchmark, might work. for the whole system is very difficult. not sure what's the best way, because results change a lot during runs
run perf measurements on a system? you need full control. what if a virus scanner starts running?

## Arne, code smells

a code smell is a "surface indication" that usually correspons to a "deeper problem" in the system - Fowler

easy to spot
not the real problem
violation or principles
missing patterns, idioms, abstractions
maintainability problem

not always a problem

example code is meant to be readable and maintainable

### long function
the deeper problem is SPR violation
Single Level of Abstraction also - different abstraction levels are mixed

blocks of code with a comment

difficult to quantify what is too long?

sieve of eratosthenes, would you cut it into pieces

tools do not replace code reviews! sometimes longer is the right solution

block comments are often good hints
reuse is not the only reason for functios

we can and should use longer func names
you might create classes and call constructors instead of simple functions

### Premature generalization
What if?
Needless or unused parameters, callbacks (always the same value...)
templates used with one type
base classes with only one derived classes

violation of KISS and YAGNI
complex design, hard to maintain
explosion of test cases or missing tests

### Deeply nested control flow
while/for/if/try-catch/...

too much to keep in mind. how did we get there what have we already checked
SRP or SLoA violation
goes with long functions

factor out functions
invert conditions for early returns

### Complicated expression

SLoA -> factor it out

how many function params do you consider too much?

lots of new code, but it also documents lots of details that won't have to be figured out

what about performance?
measure, but does it matter? + trust your profiler


### Build Smell, lack of coding

warnings
optimizers, profilers
static analysis
sanitiers


### C++ smell, no const

query functions not const
consts not declared consts...


### C++ smell, missing RAII

### C++ smell, violating rule of 5

### not using std algos, raw loops

still sometimes the loop is more readable

why there is no transform if?

ranges might be a solution

still a smell, but not every smell needs fixing

## Kevlin, FOr The Sake Of Complexity

Developers are drawn to complexity like moths to flame, often with the same outcome Neal Ford

The inherent complxity of a sw system is related to the problem it is trying to sovle
the actual complexity of a sws is related to the size and structure of the sws as built
the diff between the two is a measore of our inability to match the solution to the problem

hardware is updated, humans are not


simplicity - the ard of maximizing the amount of work not done - is essential

the number of classes does not mater, the number of relationships does
simpler in terms of what

I like to work on this? How much else should I need to know?

complex == different and connected parts. not easy to understand, complicated.

everything is a dependency, even the version of the language

the world most complicated watch -> lol, for software?

people solve problems that nobody has
speculative generality

Simplicity Before Generality
Use Before Reuse

not a repo, a time machine, a difference engine

tech debt, it is the cost owning the debt (not the cost of repaying)

> I don't lose concentration during this code

(after)noon the ninth hour from sunrise (Romans)
Mrach was the first 
seotember, october, november, december...

sweden 1712 30th february

first build something complete but simple, then add the clever parts

## guy davidson

the nature of abstraction

sub  + trahere => subtract
retract
contract
abstarct

plectere

Abstraction can be...
enxapsulation
data-hiding
the result of abstraction can be abstraction

slide 40 / problem domain defines the avstract env where the solution is developed

c++ features supporting abstractions
types
functions
  function forward declaration is hiding complexity
structs and classes 
polymorphism dynamic/static
namespaces
  do not hide complexity
  do not using using namespace
modules

the art of working at the correct level ob abstraction

options parser from the code...


Hyrum's law!
overload only on ref qualifiocation not on const! self-evident!

how you cut your talk is amazing!

## 6 ways for max
we cannot call max on two different types (int, float)

lifetime extenstion?

## Victor
we are different, often fragmented

there are obsolete information
some bad teaching materials!

### iostreams are really slow
### modules will not solve all the issues
### coroutines shipped in c++20
no libs, generators will be in 23

### printf/sprint are very fast
oh, I have to rewatch this


### C++ is not easily toolbuilder
there are tons of tools, we have to learn, think, 

### std::regex is too slow for prod use
yes. they are.
dificult to use API. both compile and runtime are slow
think about CTRE, Hana
 

### std::optional inhibits optimization and complicates APIs
it doesn't inhibit optim
don't use optional for error handling, you always want to unwrap

think about higher order functions, and monads, transformations. chains of transformations!

get inspired from other languages

it doens't complicate apis if we adapt

### names are not good
std::move does not move
std::forward does not forward
std::remove does not remove
std::function is not a function

### always pass by const&
two part ctor? one taking const& the other &&?
what happens when you have multiple params.... combinatorical problem

what about value and sink?

leverage the sink pattern! (write about the sink pattern) clang tidy modernizer
modernize-pass-by-value

### adding const always helps

well there are some places where you should not...

don't const non ref return types
don't const local values that need tale advatage on implicit move on return
don't const non-trivial value params that might need to return directly from the func
don't const any member data (breaks moves and assignment)

### all data members private??

seen as good practice

well, there are cases when structs just want to be structs

### iterators must go
plausible

### new is the enemy of old
before we had a feature, we were nonetheless able to program in c++

## Jason Turner, Making C++ Fun, Safe, Accessible

John Cage 4:33

can seomething be fun and accessible that is not safe?

someone said, something tat isfun and accessible cannot be safe

std::print with 23? fmt?

oh ranges...  

can c++ be fun?
object lifetime puzzles

jason copies everything from the compiler to the code... basic_vector<.....>....::size...

Can C++ be acessible?
for 70% it's not the first language

ftxui template usage?
gui usage template

the optimizer can optimize away a memory error release/debug
don't ship with sanitizers linked in

sticking to a specific subset of the language?
simplified lanaguage for abasic level

what's the least fun part of the language?

can we change the keywords to other langugaes? clang pass...

## Peter Mouldon, redesign
analyze the data flows
port != redesign
a design is largish tech agnostic

## Klaus
it's not a design, but an implementation pattern, as it provides no abstraction and doesn't decrease dpeendencies

# overall feedback
## difficult with the food
## great sponsorts
## many reshuffling, dynamic allocations, let's say
## everyone is so kind, even on the streets
## sponsors are not here for the last break nad/or noisy