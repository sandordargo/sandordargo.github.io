Composing function objects to fix a parameter

Partial binding

    auto plus10 = std::bind(std::plus<int>(),
                            std::placeholders::_1,
                            10);


                            https://learning.oreilly.com/library/view/the-c-standard/9780132978286/ch10.html

Predefined objects in <functional>

4 adapters
bind
mem_fn
not1
not2

calling globals
calling members
mem_fn

depreceated:
...

them vs lambdas