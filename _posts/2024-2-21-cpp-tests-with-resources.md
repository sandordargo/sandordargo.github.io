---
layout: post
title: "How to write unit tests in C++ relying on non-code files?"
date: 2024-2-21
category: dev
tags: [cpp, buildsystems, testing, filesystem]
excerpt_separator: <!--more-->
---
Recently we had a coding dojo with my colleagues where we were working on the second part of the [Racing Car Katas](https://github.com/emilybache/Racing-Car-Katas), called *TextConverter*. To sum up the problem, the `HtmlTextConverter` class takes a filename, reads the file into memory and converts its content into a not-very-sophisticated HTML text.

The goal is to test the class and potentially refactor it if you find any good reason for that. In my opinion, there are plenty of reasons to refactor this class. The main problem is that it does at least two things. It 1) reads a file and 2) converts its contents to HTML. It is difficult to write unit tests for this class because a unit test should be fast and ideally should not depend on things such as IO or network.

In this case, we clearly depend on the file system. Still, it's possible to provide a test that works. We can create a file and use it in the test. Without having that test, refactoring is not safe as we wouldn't know if we broke something.

## A first naive approach

As a first attempt, in the test directory, we created a file called `simpleText.txt` with a few lines in it and wrote this test.

```cpp
TEST(HTMLTextConverter, CorrectHtmlIsGeneratedWithSimpleNonEscapedInput) {
  const std::string expectedOutput = R"(line1<br />line2<br />line3<br />)";

  std::string filePath { "simplefile.txt" };
  HtmlTextConverter converter { filePath };
  ASSERT_EQ(expectedOutput, converter.convertToHtml());
}
```

We didn't expect it to work, but we wanted some fast feedback. It failed as the content was not read in, and the file was not found. Oh, by the way, the original code of `HtmlTextConverter` doesn't make any difference between an empty and a missing file...

As a next step, we updated the CMake settings to compile with C++17 and tried to use `std::filesystem`. We had some surprises with the filesystem API, such as its lack of support for `operator+` and the differences between `concat` and `append` or between `operator+=` and `operator/=`, but that's another story.

```cpp
TEST(HTMLTextConverter, CorrectHtmlIsGeneratedWithSimpleNonEscapedInput) {
  const std::string expectedOutput = R"(line1<br />line2<br />line3<br />)";

  std::filesystem::path filePath = std::filesystem::current_path().append("simplefile.txt");
  HtmlTextConverter converter { filePath };
  ASSERT_EQ(expectedOutput, converter.convertToHtml());
}
```
It still didn't work, the output was empty. So we decided to print the path. What could have gone wrong? It returned something unexpected to us:

```
filePath: "/Users/sandord/personal/dev/dojos/Racing-Car-Katas/Cpp/cmake-build-script/TextConverter/tests/simplefile.txt"
```
Oh la la! That's clearly not where we created the file! `cmake-build-script/` was an unexpected element of the path! Okay, so the unit test was looking for the file in the build folder, not where the file we wanted to compile originally resided...

We found three different approaches to resolve this problem.

## Use `__FILE__` to get the original path

If you want the original path of the file, you can use the [`__FILE__` preprocessor macro](https://gcc.gnu.org/onlinedocs/cpp/Standard-Predefined-Macros.html). You don't get the directory path, but the file path. This means that you have to get rid of the file name. Luckily, it's easy to do with `std::filesystem::path::remove_filename`.

```cpp
TEST(HTMLTextConverter, CorrectHtmlIsGeneratedWithSimpleNonEscapedInput) {
  const std::string expectedOutput = R"(line1<br />line2<br />line3<br />)";

  std::filesystem::path dirPath = std::filesystem::path(__FILE__).remove_filename();
  std::filesystem::path filePath =  dirPath /= "simplefile.txt";
  HtmlTextConverter converter { filePath };
  ASSERT_EQ(expectedOutput, converter.convertToHtml());
}
```

It has another downside as well, but we'll unravel that one later.

## Create a global variable from CMake

So we need the path of the original file. We know the value exactly in CMake. Also in CMake, we can create global constants. To be more precise, with `target_compile_definitions`, we can populate the `COMPILE_DEFINITIONS` property with a semicolon-separated list of preprocessor definitions using the syntax `VAR` or `VAR=value`.

Here is a way to solve our problem. We added this to our `CMakeLists.txt` file:

```py
target_compile_definitions(
  HtmlTextConverter_Test_Gmock
  PUBLIC
    RESOURCE_DIR="${CMAKE_CURRENT_SOURCE_DIR}"
)
``` 

Then we can use in the tests:

```cpp
TEST(HTMLTextConverter, CorrectHtmlIsGeneratedWithSimpleNonEscapedInput) {
  const std::string expectedOutput = R"(line1<br />line2<br />line3<br />)";

  std::filesystem::path filePath = std::filesystem::path(RESOURCE_DIR) += "/simplefile.txt";
  HtmlTextConverter converter { filePath };
  ASSERT_EQ(expectedOutput, converter.convertToHtml());
}
```

There are a couple of problems.

First, if you see this test, you have absolutely no idea where `RESOURCE_DIR` comes from. We can help a little bit on that problem, by introducing a helper variable somewhere at the beginning of the file. Another potential problem is that it's a good old char array. But with the helper variable, you solve that problem as well.

```cpp
std::filesystem::path RESOURCES_PATH {RESOURCE_DIR};

TEST(HTMLTextConverter, CorrectHtmlIsGeneratedWithSimpleNonEscapedInput) {
  const std::string expectedOutput = R"(line1<br />line2<br />line3<br />)";

  std::filesystem::path filePath = RESOURCES_PATH /= "simplefile.txt";
  HtmlTextConverter converter { filePath };
  ASSERT_EQ(expectedOutput, converter.convertToHtml());
}
```

A third problem which I already hinted about is that what if your tests change the file?

You either have to 
- undo the changes. Sure, but how? And how much work is that?
- rely on git to restore the file. Can we assume that the code is used alongside git? Maybe yes, maybe no, nevertheless it's something to consider. Let's say you can rely on Git. How much time is that going to take?
- or the best would be still to copy the file along with code files and just discard them along with the build folder. Of course, if the test files are big, this is problematic. But unit tests shouldn't depend on huge test files. Well, they shouldn't depend on text files anyhow, right...?

## Copy the resources to the build file

It's not in the scope of this article to discover handling `git` from C++ or to undo changes on a file, but we are going to see how to copy the resources to the build folder.

Copying files over to the build folder is very simple.

```py
configure_file(emptyfile.txt ${CMAKE_CURRENT_BINARY_DIR}/emptyfile.txt COPYONLY)
configure_file(escapedfile.txt ${CMAKE_CURRENT_BINARY_DIR}/escapedfile.txt COPYONLY)
configure_file(simplefile.txt ${CMAKE_CURRENT_BINARY_DIR}/simplefile.txt COPYONLY)
```

With `configure_file`, we copy our resources from the current folder (assuming that they are in the same folder as the `CMakeLists.txt` file) to the `CMAKE_CURRENT_BINARY_DIR`. In this case, we simply copy them, but [you have several options](https://cmake.org/cmake/help/latest/command/configure_file.html). This step is enough to run the tests successfully.

To have a clean solution, we need two more commands. With `add_custom_target`, we create a target to represent the copying operation. Then with `add_dependencies`, we establish the dependency relationship between this custom target and other targets in the build process. This separation of concerns allows for better organization and management of the build process in CMake.

> *It's worth noting that this doesn't make the text files part of the binary. If you have a binary to distribute that depends on text files, those files still have to be distributed along with the binary.*

```py
add_custom_target(CopyTextFile ALL DEPENDS 
                    ${CMAKE_CURRENT_BINARY_DIR}/emptyfile.txt
                    ${CMAKE_CURRENT_BINARY_DIR}/escapedfile.txt
                    ${CMAKE_CURRENT_BINARY_DIR}/simplefile.txt)    
add_dependencies(HtmlTextConverter_Test_Gmock CopyTextFile)
```

That's it. In this third solution, we copy over the files each time to the build folder, so any changes done by the tests are discarded. Besides the relative path is kept, we can use such useful constructs as `std::filesystem::current_path` in our code:

```cpp
TEST(HTMLTextConverter, CorrectHtmlIsGeneratedWithSimpleNonEscapedInput) {
  const std::string expectedOutput = R"(line1<br />line2<br />line3<br />)";

  std::filesystem::path filePath = std::filesystem::current_path().append("simplefile.txt");
  HtmlTextConverter converter { filePath };
  ASSERT_EQ(expectedOutput, converter.convertToHtml());
}
``` 

If we have another look, this code actually was one of our first, naive attempts! And eventually, the test would only pass the filename with the full path!

## Conclusion

In this article, I shared the different ways we found to make a unit test work which depends on local files. Our first solution uses the `__FILE__` macro, you have to remove the filename from the path, but it will work. The second solution will work as long as you use CMake (just like the third solution), but its readability is not the best and handling changes to the resource files might be tricky.

The third solution solves both the readability and issue and the problem of file changes in a test as it always copies the original files to the build folder. That copying might take a bit more time. Hopefully, you don't rely on huge files in your unit tests...

How do you solve this problem? Apart from removing the dependency on such files.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!