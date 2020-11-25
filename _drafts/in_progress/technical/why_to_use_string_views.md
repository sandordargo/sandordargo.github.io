What is string_view and why should we use it?


`std::string_view` has been introduced by C++17 and a typical implementation needs two information. The pointer to a character sequence and its length. The character sequence can be both a C++ or a C-string. After all, `std::string_view` is a non-owning reference to a string.

This type is needed because it's quite cheap to copy it as it only needs the above mentioned copy and their length. We can effectively use it for read operations, and besides it received the [remove_prefix](https://en.cppreference.com/w/cpp/string/basic_string_view/remove_prefix) and [remove_suffix](https://en.cppreference.com/w/cpp/string/basic_string_view/remove_suffix) modifiers.

As you might have guessed, `string_view` has these two modifiers, because it's easy to implement without modifying the underlying character sequence. Eigher the start of the character sequence or its length should be modified.
 ​
If your function doesn't need to take ownership of the `string` argument and you only need read operations (plus the above mentioned two modifier) use a `string_view` instead. In fact, `string_view` should replace all `const std::string&` parameters.

There is one drawback though. As under the hood you can either have a std::string or a std::string_view, you loose the implicait null termination. If you need that, you have to stick with std::string (const&)

The above mentioned fact that it's quite cheap to copy, and that it has one less level of pointer indirection than a `std::string&` can bring a significannt performance gain. If you're interested in the details, please read [this article](https://www.modernescpp.com/index.php/c-17-avoid-copying-with-std-string-view).



