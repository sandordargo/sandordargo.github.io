---
layout: post
title: "My late discovery of std::filesystem - Part II"
date: 2024-3-6
category: dev
tags: [cpp, filesystem, std, standardlibrary]
excerpt_separator: <!--more-->
---
Last week, [we started to discuss the main parts of `std::filesystem`](https://www.sandordargo.com/blog/2024/02/28/std-filesystem-part1-paths-and-operations) and we discovered how to work with paths, how to navigate up through the directory structure and how to move files and directories around.

This week, we are going to see how to iterate over a directory structure based on different needs and expectations.

Let's start by simply listing the contents of a single directory.

## Iterate with `directory_iterator`

Iterating over a directory and listing its items is a simple task thanks to range-based for loops and `std::filesystem::directory_iterator`. We have to pass a valid path (even in the form of a string) to the iterator and well... iterate over it.

It's also worth noting how to get only the filename as a string from the full path. From the iterator entry, we get the full path by calling `path()`, then we can get the filename by calling `filename` and then we just call the `string()` function.

```cpp
#include <fstream>
#include <iostream>
#include <filesystem>

int main() {
    std::filesystem::create_directory("temp");
    std::filesystem::create_directory("temp/subdir");
    
    // create files
    std::ofstream("temp/file1.txt").put('a');
    std::ofstream("temp/file2.txt").put('b');
    std::ofstream("temp/subdir/file3.txt").put('c');

    std::cout << "The contents of temp:\n";
    for (const auto& entry : std::filesystem::directory_iterator("temp")) {
        const auto filename = entry.path().filename().string();
        if (entry.is_directory()) {
            std::cout << "directory: " << filename << '\n';
        }
        else if (entry.is_regular_file()) {
            std::cout << "file: " << filename << '\n';
        }
        else {
            std::cout << "unknown type: " << filename << '\n';
        }
    }
}
/*
The contents of temp:
directory: subdir
file: file1.txt
file: file2.txt
*/
```

The above example only lists the contents of one single directory, it doesn't get into the subfolders recursively. In order to do so, we have to extract the iteration into its own method and call it with a subfolder path whenever we find a subfolder. Notice that in order to print the contents in a tree-like structure, we keep track of the level or depth of the path.

```cpp
#include <fstream>
#include <iostream>
#include <filesystem>

void iterateOverDirectory(const std::filesystem::path& root, int level = 0) {
    for (const auto& entry : std::filesystem::directory_iterator(root)) {
        const auto filename = entry.path().filename().string();
        if (entry.is_directory()) {
            std::cout << std::setw(level * 3) << "" << "directory: " <<filename << '\n';
            iterateOverDirectory(entry, level + 1);
        }
        else if (entry.is_regular_file()) {
            std::cout << std::setw(level * 3) << "" << "file: " << filename << '\n';
        }
        else {
            std::cout << std::setw(level * 3) << "" << "unknown type: " << filename << '\n';
        }
    }
}

int main() {
    std::filesystem::create_directory("temp");
    std::filesystem::create_directory("temp/subdir");
    
    // create files
    std::ofstream("temp/file1.txt").put('a');
    std::ofstream("temp/file2.txt").put('b');
    std::ofstream("temp/subdir/file3.txt").put('c');

    std::cout << "The contents of temp:\n";
    iterateOverDirectory("temp");
}
/*
The contents of temp:
directory: subdir
   file: file3.txt
file: file1.txt
file: file2.txt
*/
```

But there is a simpler solution!

## Iterate with `recursive_directory_iterator`

We can use the `recursive_directory_iterator`, hide the recursive calls and get all the paths.

```cpp
#include <fstream>
#include <iostream>
#include <filesystem>

void iterateOverDirectory(const std::filesystem::path& root) {
    for (const auto& entry : std::filesystem::recursive_directory_iterator(root)) {
        std::cout << entry << '\n';
    }
}

int main() {
    std::filesystem::create_directory("temp");
    std::filesystem::create_directory("temp/subdir");
    
    // create files
    std::ofstream("temp/file1.txt").put('a');
    std::ofstream("temp/file2.txt").put('b');
    std::ofstream("temp/subdir/file3.txt").put('c');

    iterateOverDirectory("temp");
}
/*
"temp/subdir"
"temp/subdir/file3.txt"
"temp/file1.txt"
"temp/file2.txt"
*/
```

If we need access to the depth and display a directory tree, we can still do that with `recursive_directory_iterator`, but we cannot use a range-based for loop.

```cpp
#include <fstream>
#include <iostream>
#include <filesystem>

void iterateOverDirectory(const std::filesystem::path& root) {
    for (auto entry = std::filesystem::recursive_directory_iterator(root); entry != std::filesystem::recursive_directory_iterator(); ++entry) {
        const auto filename = entry->path().filename().string();
        const auto level = entry.depth();
        if (entry->is_directory()) {
            std::cout << std::setw(level * 3) << "" << "directory: " <<filename << '\n';
        }
        else if (entry->is_regular_file()) {
            std::cout << std::setw(level * 3) << "" << "file: " << filename << '\n';
        }
        else {
            std::cout << std::setw(level * 3) << "" << "unknown type: " << filename << '\n';
        }
    }
}

int main() {
    std::filesystem::create_directory("temp");
    std::filesystem::create_directory("temp/subdir");
    
    // create files
    std::ofstream("temp/file1.txt").put('a');
    std::ofstream("temp/file2.txt").put('b');
    std::ofstream("temp/subdir/file3.txt").put('c');

    iterateOverDirectory("temp");
}
/*
directory: subdir
   file: file3.txt
file: file1.txt
file: file2.txt
*/
```

The reason behind is that the `value_type` of `recursive_directory_iterator` is `std::filesystem::directory_entry` which has no notion of depth. By using the range-based version, we lose access to the iterator type.

## Skip certain files and folders

If we want to skip certain extensions, we can have a guard clause at the beginning of the body of the loop. If we use a regular iterator, where we take care of going into the subfolders, we also have to make sure that we don't skip directories just because they don't match the allowed extensions. 

```cpp
#include <algorithm>
#include <array>
#include <fstream>
#include <iostream>
#include <filesystem>

std::array<std::string, 2> allowed_extensions {".h", ".cpp"};

void iterateOverDirectory(const std::filesystem::path& root, int level = 0) {
    for (const auto& entry : std::filesystem::directory_iterator(root)) {
        const auto filename = entry.path().filename().string();
        const auto extension = entry.path().extension().string();
        if (!entry.is_directory() && std::ranges::find(allowed_extensions, extension) == allowed_extensions.end()) {
            continue;
        }
        if (entry.is_directory()) {
            std::cout << std::setw(level * 3) << "" << "directory: " <<filename << '\n';
            iterateOverDirectory(entry, level + 1);
        }
        else if (entry.is_regular_file()) {
            std::cout << std::setw(level * 3) << "" << "file: " << filename << '\n';
        }
        else {
            std::cout << std::setw(level * 3) << "" << "unknown type: " << filename << '\n';
        }
    }
}

int main() {
    std::filesystem::create_directory("temp");
    std::filesystem::create_directory("temp/include");
    std::filesystem::create_directory("temp/src");
    
    // create files
    std::ofstream("temp/CMakeLists.txt").put('a');
    std::ofstream("temp/include/moreinfo.txt").put('a');
    std::ofstream("temp/include/moduleA.h").put('a');
    std::ofstream("temp/include/moduleB.h").put('b');
    std::ofstream("temp/src/moduleA.cpp").put('a');
    std::ofstream("temp/src/moduleB.cpp").put('b');

    iterateOverDirectory("temp");
}
/*
directory: include
   file: moduleB.h
   file: moduleA.h
directory: src
   file: moduleA.cpp
   file: moduleB.cpp
*/
```
In fact, if you want to print directory names like in the above example, then even in the recursive version you should keep the `!entry.is_directory()` part of the guard clause. But even if you remove it, all the subdirectories will still be searched.

To skip certain folders, let's add another guard clause in which we check if an entry is a directory and if it matches the name "build" as in our next example, we want to avoid listing the contents of `build` folders.

```cpp
#include <algorithm>
#include <array>
#include <fstream>
#include <iostream>
#include <filesystem>

std::array<std::string, 2> allowed_extensions {".h", ".cpp"};

void iterateOverDirectory(const std::filesystem::path& root, int level = 0) {
    for (const auto& entry : std::filesystem::directory_iterator(root)) {
        const auto filename = entry.path().filename().string();
        const auto extension = entry.path().extension().string();
        if (!entry.is_directory() && std::ranges::find(allowed_extensions, extension) == allowed_extensions.end()) {
            continue;
        }
        if (entry.is_directory() && filename == "build") {
            continue;
        }
        if (entry.is_directory()) {
            std::cout << std::setw(level * 3) << "" << "directory: " <<filename << '\n';
            iterateOverDirectory(entry, level + 1);
        }
        else if (entry.is_regular_file()) {
            std::cout << std::setw(level * 3) << "" << "file: " << filename << '\n';
        }
        else {
            std::cout << std::setw(level * 3) << "" << "unknown type: " << filename << '\n';
        }
    }
}

int main() {
    std::filesystem::create_directory("temp");
    std::filesystem::create_directory("temp/build");
    std::filesystem::create_directory("temp/include");
    std::filesystem::create_directory("temp/src");
    
    // create files
    std::ofstream("temp/CMakeLists.txt").put('a');
    std::ofstream("temp/build/moduleA.h").put('a');
    std::ofstream("temp/build/moduleA.cpp").put('b');
    std::ofstream("temp/include/moreinfo.txt").put('a');
    std::ofstream("temp/include/moduleA.h").put('a');
    std::ofstream("temp/include/moduleB.h").put('b');
    std::ofstream("temp/src/moduleA.cpp").put('a');
    std::ofstream("temp/src/moduleB.cpp").put('b');

    iterateOverDirectory("temp");
}
```

This approach will not work if you want to use a recursive iterator, we'd only avoid printing *directory: build*, but we'd list it's contents. We have to disable recursing for that folder and we can do that by calling `std::filesystem::recursive_directory_iterator::disable_recursion_pending`.

```cpp
#include <algorithm>
#include <array>
#include <fstream>
#include <iostream>
#include <filesystem>

std::array<std::string, 2> allowed_extensions {".h", ".cpp"};

void iterateOverDirectory(const std::filesystem::path& root) {
    for (auto entry = std::filesystem::recursive_directory_iterator(root); entry != std::filesystem::recursive_directory_iterator(); ++entry) {
        const auto filename = entry->path().filename().string();
        const auto extension = entry->path().extension().string();
        const auto level = entry.depth();
        if (!entry->is_directory() && std::ranges::find(allowed_extensions, extension) == allowed_extensions.end()) {
            continue;
        }
        if (entry->is_directory() && filename == "build") {
            entry.disable_recursion_pending();
            continue;
        }
        if (entry->is_directory()) {
            std::cout << std::setw(level * 3) << "" << "directory: " <<filename << '\n';
        }
        else if (entry->is_regular_file()) {
            std::cout << std::setw(level * 3) << "" << "file: " << filename << '\n';
        }
        else {
            std::cout << std::setw(level * 3) << "" << "unknown type: " << filename << '\n';
        }
    }
}

int main() {
    std::filesystem::create_directory("temp");
    std::filesystem::create_directory("temp/build");
    std::filesystem::create_directory("temp/include");
    std::filesystem::create_directory("temp/src");
    
    // create files
    std::ofstream("temp/CMakeLists.txt").put('a');
    std::ofstream("temp/build/moduleA.h").put('a');
    std::ofstream("temp/build/moduleA.cpp").put('b');
    std::ofstream("temp/include/moreinfo.txt").put('a');
    std::ofstream("temp/include/moduleA.h").put('a');
    std::ofstream("temp/include/moduleB.h").put('b');
    std::ofstream("temp/src/moduleA.cpp").put('a');
    std::ofstream("temp/src/moduleB.cpp").put('b');

    iterateOverDirectory("temp");
}
/*
directory: include
   file: moduleB.h
   file: moduleA.h
directory: src
   file: moduleA.cpp
   file: moduleB.cpp
*/
```

It's up to you which approach you like better. Probably this latter one is a bit more neat, but the name `disable_recursion_pending` might give you some extra time to figure out what is going on.

## Conclusion

In this article, continued learning about `std::filesystem`. This week we targeted one single topic, how to iterate over the files of a directory or recursively over a whole directory structure. We also saw how to skip files with certain extensions or even directories with certain names.

What's your favourite way of iterating over directories in C++?

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!