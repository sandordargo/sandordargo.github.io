
## Walter Brown
dedicated to his son and daughter in law

Alex Stepanov 

## Clare Macrae

Approval tests


https://github.com/approvals/ApprovalTests.cpp

Tests logs/output
Tests files
pics

## Iterators - to ranges

shows certain limitations (iterators, rvalue to filters)
think cell implementation:
	https://github.com/think-cell/range


## Matt Godbolt

strong value types / tiny types
alias/typedef, not really
class, YES!

explicit, expicit, explicit!

is there a way to turn the comment into an actionable check
assert size!

char sized enum class 
enum class MessageType : char {
	Add = 'A';
	Modify = 'D';
	Delete = 'M';
}

don't use default, if you add a new one....

[[no_discard]]

separate concerns => remove aplogetic comments



sometimes you cannot call a function twice. what to do?
rvalue qualifier to add to the function.
std::move(compiler).compile()

let the compiler help you



comment smells
warning
strong value types
RAII/CADR

SW is data manipulation
code should minimuse usage of slow things, mem access, branch misses, instruction fetching

Build a data model, so that things are were you expect it
what is span?
gsl::span
contiguous memory, small easy and fast to move around

each vector is dyn allocated, one vec is here, other vec is there
when you move from one to another, cache miss!

spans are closer to each other

very low level...
insturction locality

nothing is better until you measure based on your parameters (perf, memory, cache misses)

there is a cost of optimization in terms of maintainabilty and readaility


---
KonMari your code
Gather it,
Does it spark joy? Do you need it?

Thanks dor the code, you delete.

Choose a place ad toght technique


Tina: KONMARI, ISO Clar


Hidden languages of C++
OO C++
Template C++
,,
4

There are more:
Cmake, Bazel, Makefiles, etc
C preprocessor
C++ versions
	int* C++17: I also like to live dangerously
Error Messages


Fred Tingaud
we come here for new patterns, but we have old code
how can we modernize

vacuum cleaner,  we'd have more time. no. we redefined what we expect from a carpet. we also had more carpet

boy scout rule!
golden boy refactoring :D

repetitive, error-prone, whack a mole

AST

clang-tidy => range based
 => auto ptr

 registerMatcher
 check


 trivia: replace && with and and rvalue references still compile

 check Demeule's video
 steveire.wordpress.com
 gdbolt / z/ clang-query


 On understanding object orientation revisited


 Kate Gregory
 nomenclature -> stay consistent, don't use the same name for different things
 business should name the things

 don't mismatch natural pairs
 no overloaded terms

 verbs for functions
 [[nodiscard]] -> your name is wronf
 class are nouns er(manager, updater) are suspicious
 don't overdecorate

 use adjectives
 don't encode types
 don't use type names as names (when use auto, you'll have better names)

 think about the greppers

 Jon Kalb - OOP

 programming paradign using polymorphism based on runtume function dispatch using virtual functions

 we use OOP, how do it best

 follow semantics, not just syntax

 inheritance is for well defined abstractions
 not for code reue
 is-a
 [misspoke]

 we only dereference to get something from the base class (animal/lizard)
 make non-leaf abstract to have better hierachies
 	1)either base-only or leaf
 	2)make base abstract with protected assignment
 	3)leaf concrete, public assignment, and final!

NVI idiom, non-virtual public api -> articles
	base class enforces pre/post ocnditions,nstrumentations
	robust in face of changes
	each interface can take its natural shape

interface for callers and derives

design idioms:
is-a
non-leaf abstract
NVI


Herb
if a new thing is available on the old thing,it's gonna be easier to have it use