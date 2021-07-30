A good API, a good library is easy to use and hard to misuse. As we saw, `std::random_shuffle` might be easy to use, but it's just as easy to misuse. `std::shuffle` takes standard generators as those defined in <random>. To shuffle the elements of the range without such a generator. They are not top




	If you look at the include statements, you can observe that we are using legacy headers (`<ctime>` and `<cstdlib>` start with C) and the third (optional) argument.

At 1), from `<ctime>` we use `std::time` in order to provide a seed for `<cstdlib>`'s `std::srand`. Then in 2) we simply have to pass any function as a third - optional - parameter to `std::random_shuffle`. Yes, any. You could simply pass a function always returning 42. Would it work well? Try and see!