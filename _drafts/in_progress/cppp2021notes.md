
## GOdbolt

superpower
ubiquity
performance
multi paradigm
clear object lifetime

:(
UB
wrong defaults
legcy support
confusign syntax

not all cprogram is a valid c++ program (var name class)

CEDD  compiler error driven development

string capitalize with range! that's nice

use sanitizers!
small steps and CEDD

easier to find and fix bugs with modern code
it's more understandable
it keeps coders happy (bus forget about this)


sommerlad

phylosophy:
less code = more software

more structs?

roles?
value - what 
subject - here
relation - where

manager to cleanup

value semantics
copyability

never define a dtor with an empty body

-----

discussion
awesome, probably even easier to jump in
agile, it's not their job to decide what is good


joly/misra

why hated... old, return, good examples

explicit is better then terse

the goal of misra, not to ban you, but to make you valdate
exception: mandatory rules


barry
- push and pull based iterators
- internal vs external iteration


---
philippe bourgeau:
  solution focus workshop, activity

  
damien tipi
CPP is green


sy
new ways of seeing code, seeing art
why complain about why males?
greeks => beauty = virtue

you cannot separate aesthetics from the cultural

what is art?
who do i tell if this is art or not?

is code art? is it a meaningful question?

what is beautiful code?

if we make our code beautiful, it becomes better, easier to maintain and read

y combinator to turna lambda into a recursive

is beauty contextual?

code as poetry
code as art, interesting!

mattina

see code as art and move from one language to another

knowing the context changes what you think about code

beauty in skill
beauty is essential, but subjective

art is functioning cognitively, aesthetically

can i experience code as art?
can my code reflect me? (yes, KG)
can code be beautiful? yes

speak about greek aestathitcs

arthur boyd, water painting

My favorite quote, linking beauty and efficiency : 
"How do we convince people that in programming simplicity and clarity -- in short: what mathematicians call 'elegance' -- are not a dispensable luxury, but a crucial matter that decides between success and failure?" Edsger W. Dijkstra

around 120 ppl at the conference


Sebastian Gonzalve
on refactoring

If you do not have tests, your priority is to have them, not to change the code.

more work, but enforce the use of correct APIs, more work in short, but then no maintenance

FearOfFailure


Train the team!

pointer, but no copy ctor, no deep copy, holy crap


performance of virtual functions

application specialist
https://johnysswlab.com/


they are usually not tested in relevant ways

vfunc address isnot known at compile time, it has be looked up at runtime

there is always a vtable

a vector or 20 mill obj of same time
20 million calsl to the vf vs 20 million calls to the non virt function, no big overhead for big funcs, for smalls, there is some (18%)

small programs, vector of objects vs vector of ptr, not that bad

vector of objs, no problem od cache misses

the problem is not with the virtual function! the problem is that you need stuff on the heap! 7 times slower

use a variant wih visitor

vfuncs cannot be inlined, no optimizatoion!

=> type based processing

jump destination guessing (30:20)

instruction cache eviction - 35
with different implementation, no hot code

not a problem with vfuc, it's the environment! predictability

data oriented desygn -> type based processing

data cache misses!
no vector of pointers!

always focus on the hot code

cache misses
intel vtune profiler
https://www.intel.com/content/www/us/en/developer/tools/oneapi/vtune-profiler-download.html?operatingsystem=linux&distributions=aptpackagemanager
pmu tools