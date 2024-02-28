---
layout: post
title: "Late discovery of std::filesystem - Part I"
date: 2024-1-3
category: dev
tags: [cpp, filesystem, std, standardlibrary]
excerpt_separator: <!--more-->
---
I know that this is not a new topic at all. But this blog in its roots is to document what I learn, and I haven't used the filesystem library up until a few weeks ago. After the initial encounter, I deliberately spent a bit more time exploring it and I want to share what I found.

I don't want to go over the C++ Reference documentation and I also don't want to simply repeat what [Bartek already shared here](https://www.cppstories.com/2017/08/cpp17-details-filesystem/).

I rarely use C++ to manipulate the filesystem. That usually comes up with Python. So I decided to go through my Python use cases and see how that would in C++ with the std::filesystem library. Which was introduced in C++17, but its roots are back in the [boost::filesystem](https://www.boost.org/doc/libs/1_84_0/libs/filesystem/doc/index.htm) library.

## Get the current filename

By current file, I don't mean the executable file, but the source code file. This is not something that we are going to achieve with `std::filesystem`. You either need to use the standard `__FILE__` macro, or you need to use `std::source_location::current()::file_name`. Let's see a little example:

```cpp
std::cout << __FILE__ << '\n';
std::cout << std::source_location::current().file_name() << '\n'; // requires C++20
```
Even though we don't do this with `std::filesystem`, I wanted to include it, because requiring the current filename is often needed for the following use cases.

## Get the directory of a file

Now that we have the absolute path of the current file, let's see how to get the directory of it.

Nothing is simpler than that!

### Get the absolute path

First of all, if we need the current directory, we can use `std::filesystem::current_path()`. But if that's not the case and we already have an absolute path of a file anywhere even just as a string, we can use `remove_filename()`.

```cpp
#include <iostream>
#include <filesystem>

int main() {
    std::cout << std::filesystem::current_path() << '\n';
    std::cout << __FILE__ << '\n';
    std::cout << std::filesystem::path(__FILE__).remove_filename() << '\n';
}
/*
"/app"
/app/example.cpp
"/app/"
*/
```

### Get the relative path

You might say that you are not interested in the absolute path, but rather in the relative. Let's assume that we have a file called `/Users/sandor/personal/dev/dojos/Racing-Car-Katas/Cpp/TextConverter/tests/HtmlTextConverter_Test.cpp`. Your first idea might be to try `std::filesystem::path::relative_path()` but then you realize that it doesn't take any parameters! What does relative mean then? Well, it only removes the prefix that signals the root, e.g. `/` or `C:\`...

```cpp
#include <iostream>
#include <filesystem>

int main() {
    std::filesystem::path p("/Users/sandor/personal/dev/dojos/Racing-Car-Katas/Cpp/TextConverter/tests/HtmlTextConverter_Test.cpp");
    std::cout << p << '\n';
    std::cout << p.relative_path() << '\n';
}
/*
"/Users/sandor/personal/dev/dojos/Racing-Car-Katas/Cpp/TextConverter/tests/HtmlTextConverter_Test.cpp"
"Users/sandor/personal/dev/dojos/Racing-Car-Katas/Cpp/TextConverter/tests/HtmlTextConverter_Test.cpp"
*/
```

If you want to get the relative path compared to another path, then instead of the above member function we have to use a free function called `std::filesystem::relative`.

```cpp
#include <iostream>
#include <filesystem>

int main() {
    std::filesystem::path filePath("/Users/sandor/personal/dev/dojos/Racing-Car-Katas/Cpp/TextConverter/tests/HtmlTextConverter_Test.cpp");
    std::filesystem::path repoRoot("/Users/sandor/personal/dev/dojos/Racing-Car-Katas/");
    std::cout << std::filesystem::relative(filePath, repoRoot) << '\n';
    std::cout << std::filesystem::relative(repoRoot, filePath) << '\n';
}
/*
"Cpp/TextConverter/tests/HtmlTextConverter_Test.cpp"
"../../../../"
*/
```

As we can see, the only question is which path goes first. The correct way to read this is _"give me the relative path of the left-hand path compared to the right-hand path"_.

In my opinion, a nice API could have used a member function, such as `std::filesystem::path::relative_to(otherPath)`. But there must have been other considerations that I'm not aware of.

## Step up a directory

If you have a path at your hands and you want the path of the parent directory, you don't have a difficult task, just use the `parent_path()` member function.

```cpp
#include <iostream>
#include <filesystem>

int main() {
    std::filesystem::path p("/Users/sandor/personal/dev/dojos/Racing-Car-Katas/Cpp/TextConverter/tests/HtmlTextConverter_Test.cpp");
 
    while (p != p.root_directory()) {
        std::cout << p.parent_path() << '\n';
        p = p.parent_path();
    }

}

/*
"/Users/sandor/personal/dev/dojos/Racing-Car-Katas/Cpp/TextConverter/tests"
"/Users/sandor/personal/dev/dojos/Racing-Car-Katas/Cpp/TextConverter"
"/Users/sandor/personal/dev/dojos/Racing-Car-Katas/Cpp"
"/Users/sandor/personal/dev/dojos/Racing-Car-Katas"
"/Users/sandor/personal/dev/dojos"
"/Users/sandor/personal/dev"
"/Users/sandor/personal"
"/Users/sandor"
"/Users"
"/"
*/
```

## Check if a file is a file or directory

Let's check now if a file or directory exists. To do so, we leave behind the RacingKar katas and we are going to use the standard library to create some files and directories. With `std::filesystem::create_directory`, it's easy to create a new folder. With `std::ofstream` we can create a file(stream) and with `std::ofstream::put` we can add a character to it. 

```cpp
#include <filesystem>
#include <fstream>
#include <iostream>
#include <string>


int main() {
    std::cout << std::boolalpha;
    std::string newDirName {"temp"};
    std::cout << newDirName << " exists? " << std::filesystem::exists(newDirName) << '\n';
    std::filesystem::create_directory("temp");
    std::cout << newDirName << " exists? " << std::filesystem::exists(newDirName) << '\n';
    std::cout << newDirName << " is directory? " << std::filesystem::is_directory(newDirName) << '\n';

    std::filesystem::path newFilePath{"temp/file1.txt"};
    // create a file
    std::ofstream(newFilePath).put('a');

    std::cout << newFilePath << " exists? " << std::filesystem::exists(newFilePath) << '\n';
    std::cout << newFilePath << " is directory? " << std::filesystem::is_directory(newFilePath) << '\n';
    std::cout << newFilePath << " is block file? " << std::filesystem::is_block_file(newFilePath) << '\n';
    std::cout << newFilePath << " is charachter file? " << std::filesystem::is_character_file(newFilePath) << '\n';
    std::cout << newFilePath << " is regular file? " << std::filesystem::is_regular_file(newFilePath) << '\n';
}

/*
temp exists? false
temp exists? true
temp is directory? true
"temp/file1.txt" exists? true
"temp/file1.txt" is directory? false
"temp/file1.txt" is block file? false
"temp/file1.txt" is charachter file? false
"temp/file1.txt" is regular file? true
*/
```

As you can see, by calling `std::filesystem::exists(path)`, it's easy to check whether a file or directory exists and there are different additional query functions available to check if a file object is a directory, a block, character or regular file. These file types are defined in the POSIX standard.

## Copy or rename a file

In the next example, after creating some files and directories, we are going to first copy a directory, then some files and then we will rename a file. To facilitate our example, we'll also use `std::filesystem::remove` to remove directories and files.

```cpp
#include <filesystem>
#include <fstream>
#include <iostream>
#include <string>


int main() {
    std::cout << std::boolalpha;
    std::string newDirName {"temp"};
    std::filesystem::create_directory("temp");
    
    std::filesystem::path newFilePath{"temp/file1.txt"};
    std::filesystem::path anotherNewFilePath{"temp/file2.txt"};
    // create a file
    std::ofstream(newFilePath).put('a');
    std::ofstream(anotherNewFilePath).put('b');

    std::cout << "========= copy a dir ===========\n";
    std::filesystem::path anotherDir {"anotherTemp"};
    std::cout << "anotherTemp" << " exists? " << std::filesystem::exists(anotherDir) << '\n';
    std::filesystem::copy(newDirName, anotherDir);
    std::cout << "anotherTemp/file1.txt" << " exists? " << std::filesystem::exists("anotherTemp/file1.txt") << '\n';
    std::cout << "anotherTemp/file2.txt" << " exists? " << std::filesystem::exists("anotherTemp/file2.txt") << '\n';
    

    std::cout << "========= copy a file to...? ===========\n";
    std::filesystem::path yetAnotherDir {"yetAnotherTemp"};
    std::cout << "yetAnotherTemp" << " exists? " << std::filesystem::exists(yetAnotherDir) << '\n';
    std::filesystem::copy(newFilePath, yetAnotherDir);
    std::cout << "yetAnotherTemp/file1.txt" << " exists? " << std::filesystem::exists("yetAnotherTemp/file1.txt") << '\n';
    std::cout << "yetAnotherTemp/file2.txt" << " exists? " << std::filesystem::exists("yetAnotherTemp/file2.txt") << '\n';
    std::cout << "yetAnotherTemp" << " exists? " << std::filesystem::exists(yetAnotherDir) << '\n';
    std::cout << "yetAnotherTemp" << " is directory? " << std::filesystem::is_directory(yetAnotherDir) << '\n';
    std::cout << "yetAnotherTemp" << " is regualr file? " << std::filesystem::is_regular_file(yetAnotherDir) << '\n';

    std::cout << "========= copy files ===========\n";
    std::filesystem::remove(yetAnotherDir);
    std::cout << "yetAnotherTemp" << " exists? " << std::filesystem::exists(yetAnotherDir) << '\n';
    // std::filesystem::copy(newFilePath, "yetAnotherTemp/file1.txt"); // ERROR, target directory does not exist
    std::filesystem::create_directory(yetAnotherDir);
    std::filesystem::copy(newFilePath, "yetAnotherTemp/file1.txt");
    std::cout << "yetAnotherTemp/file1.txt" << " exists? " << std::filesystem::exists("yetAnotherTemp/file1.txt") << '\n';
    
    std::filesystem::copy_file(anotherNewFilePath, "yetAnotherTemp/file2.txt");
    std::cout << "yetAnotherTemp/file2.txt" << " exists? " << std::filesystem::exists("yetAnotherTemp/file2.txt") << '\n';
    
    std::cout << "========== rename ==========\n";
    std::filesystem::rename("yetAnotherTemp/file2.txt", "yetAnotherTemp/file2R.txt");
    std::cout << "yetAnotherTemp/file2.txt" << " exists? " << std::filesystem::exists("yetAnotherTemp/file2.txt") << '\n';
    std::cout << "yetAnotherTemp/file2R.txt" << " exists? " << std::filesystem::exists("yetAnotherTemp/file2R.txt") << '\n';
}
/*
========= copy a dir ===========
anotherTemp exists? false
anotherTemp/file1.txt exists? true
anotherTemp/file2.txt exists? true
========= copy a file to...? ===========
yetAnotherTemp exists? false
yetAnotherTemp/file1.txt exists? false
yetAnotherTemp/file2.txt exists? false
yetAnotherTemp exists? true
yetAnotherTemp is directory? false
yetAnotherTemp is regualr file? true
========= copy files ===========
yetAnotherTemp exists? false
yetAnotherTemp/file1.txt exists? true
yetAnotherTemp/file2.txt exists? true
========== rename ==========
yetAnotherTemp/file2.txt exists? false
yetAnotherTemp/file2R.txt exists? true
*/
```
In the "copy a file to...?" part, we can observe that when we want to copy a file, the destination is not the destination directory, but the destination path. If you don't pay attention, you might copy a file to a place which was meant to be a directory.

But it can only happen if you haven't created the destination directory yet. It's important to note, that you cannot use `std::filesystem::copy` to copy to a non-existing directory. You have to make sure that it exists. Even though as a third, optional parameter, it takes `std::filesystem::copy_options`, seemingly [there is no option to create automatically the needed directory](https://en.cppreference.com/w/cpp/filesystem/copy_options).

While `std::filesystem::copy` can be used to copy both files and directories, there is also `std::filesystem::copy_file` which can only copy a single file. Its name is more expressive, but that's not the only difference. While `copy` is a void function, `copy_file` returns a boolean to show if a copy was successful (`true`) or not (`false`).

## Conclusion

Today, I shared with you a part of what I learned about the `std::filesystem` library. I must tell you that I found it pretty usable, despite the fact that sometimes I could have imagined a more intuitive API. We had a look into how to navigate up on a path, how to remove a filename from it and also we also checked how to copy, rename or delete files.

Next week, we are going to discuss how to iterate over a directory structure, a quite common operation when you have to apply some changes to a whole repository. Stay tuned!

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!