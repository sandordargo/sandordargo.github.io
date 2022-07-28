Bill Weinman has been involved in technology since he built his first computer at age 16,
in 1971. He's been coding in C and C++ since the early 1970s 
really?

the entire C language - no

Who this book is for
This book is for intermediate to advanced C++ programmers who want to get more out of
the C++20 Standard Template Library. Basic knowledge of coding and C++ concepts are
necessary to get the most out of this book.


You can align values, left ( < ), right ( > ), or center ( ^ ), with or without a fill character:
format("{:.<10}", ival);
format("{:.>10}", ival);
format("{:.^10}", ival);
// 42........
// ........42
// ....42....
• You can set the decimal precision of values:
format("π: {:.5}", pi);
// π: 3.1416
so depending on what comes after ., . means a different thing. it's NOT very intuitive...

print is coming in c++23, isn't it

The C++20 format library solves a long-standing problem by providing a type-safe text
formatting library that is both efficient and convenient.



But if you try to use the result in a run-time context, you will get an error about memory
that was allocated during constant evaluation:
int main() {
constexpr auto vec = use_vector();
return vec[0];
}
This is because the vector object was allocated and freed during compilation. So, the object
is no longer available at run time.


Because the size() method is constexpr -qualified, the expression can be evaluated at
compile time.




spaceship operator, can be used directly for comparisons with PODs or to delcare compariosn operators
strong/partial ordering

concepts name don't follow the standard
any boolean expression!
no, not AND / OR
not!


vector<int> vi { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };
auto even = [](int i) { return 0 == i % 2; };
auto x2 = [](auto i) { return i * 2; };
auto tview = vi | views::filter(even) | views::transform(x2);
why int, why auto


spans are the views for arrays, like string_view
it doesn't own data

Use structured binding to return multiple
this book is clearly not about C++20 STL
speaks a lot of c++17 features and also about non stl features (chapter 2)

template argument deduction for classes since cpp17, make_pair, etc is obsolete. what about make_unique?

The deque (commonly pronounced, deck) is a double-ended queue. It's a
sequential container that can be expanded or contracted on both ends. A deque
allows random access to its elements, much like a vector , but does not guarantee
contiguous storage.

map, unordered map, could be more precise


quick delete is an interesting concept

emplace vs try_emplace

Use set to sort and filter user input