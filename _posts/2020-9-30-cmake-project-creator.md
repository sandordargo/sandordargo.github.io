---
layout: post
title: "Cmake Project Creator at your disposal"
date: 2020-9-30
category: dev
tags: [cpp, cmake, showdev, hacktoberfest]
excerpt_separator: <!--more-->
---
After [Daily C++ Interview](https://www.dailycppinterview.com/) that I introduced in early September, let me share with you another project I have been working on for in my learning time for the last couple of months. [Cmake Project Creator](https://github.com/sandordargo/cmake-project-creator) is fully open source, and if you are looking forward to [Hacktoberfest](https://hacktoberfest.digitalocean.com/), this might be interesting for you in case you speak Python and interested in C++.
<!--more-->

## What is Cmake Project Creator about?

As a C++ developer, have you had the problem with starting new projects because you don't know enough about how to compile multiple files or directories together? Or you simply didn't have the time in the scope of a small coding dojo to spend 10-15 minutes to set up a project with unit tests running?

The former one was definitely a problem to me and the latter was also a recurring problem at coding dojos. When you only have so much time for a kata, these minutes count.

[Cmake Project Creator](https://github.com/sandordargo/cmake-project-creator) is one of the possible solutions. It helps you generate a new C++ project. Instead of writing all the CMakeLists and create all the folders by hand, you can simplify this to run the tool with a shipped set-up or you can write a description by yourself.

## What does it generate?

[Cmake Project Creator](https://github.com/sandordargo/cmake-project-creator) creates a directory structure for you with all the required `CMakeLists.txt` files, and if specified, all the plumbing to include dependencies from [`Conan`](https://conan.io/) - such as GTest. Besides, you'll get a skeleton of a class in all components, and in case tests were required, a failing unit test will be generated along with an easy way to invoke it.

First, why a failing unit test? When I create a new component, I always write a failing assertion such as `ASSERT_EQ(1, 2)` to verify that I see the expected failure. Then I fix it so that I can see it's working.

What you get after generating your project is the failing test.

And what's the easy way to invoke it?

[Cmake Project Generator](https://github.com/sandordargo/cmake-project-creator) also creates a small script called `runTests.sh`, which cleans up the results of the previous build, fetches the external dependencies, compiles the project, and runs all the generated unit tests.

## How to use it?

I won't go into details on how to write a descriptor file, you can check the [README for that](https://github.com/sandordargo/cmake-project-creator), but briefly, it's a `JSON` file that can be quite simple for a simple project and it might get a bit more complex, but I think it's still easier than writing all the needed files by hand.

To give you an example, if you want to generate a project with a single component and with unit test, in other words, if you want the below structure, you can invoke the tool with the parameter `-s single`.

```
myProject
|_ include
|_ src
|_ test

``` 

`-s single` instructs the tool to create a project with one include, one source, and one test folder. GTest will be included for unit testing through Conan.

But how that `single` descriptor looks like. It looks like this:

```json
{
  "projectName": "MyTestProjectSingle",
  "directories": [{
      "name": "src",
      "type": "source",
      "library": null,
      "executable": "true",
      "include": "true",
      "dependencies": [],
      "subdirectories": []
    },
    {
      "name": "include",
      "type": "include",
      "subdirectories": []
    },
    {
      "name": "tests",
      "type": "tests",
      "dependencies": [{
        "type": "conan",
        "name": "gtest",
        "version": "1.8.1"
      }],
      "subdirectories": []
    }
  ]
}
```

If you want a deeper explanation of what each field means, check out the documentation, especially that it might change and docs will be updated, not this article. This example is here only to represent the simplicity of writing such a description - compared to write all the CMakefiles and Conan configurations by hand.

## How can you contribute?

You can find the project on [Github](https://github.com/sandordargo/cmake-project-creator) and feel free to check the [issues tab](https://github.com/sandordargo/cmake-project-creator/issues). In fact, before you start working on something, I'd urge you to check the issues tab to make sure that the problem/new feature is already covered with a ticket and nobody took it yet.

That's a way to discuss whether the requested change is in scope and to avoid that you work in vain as someone already started the implementation of the same fix or enhancement.

We use Python 3.8 as a programming language and obviously, you need some familiarity with C++, but mostly with [CMake](https://cmake.org/). For unit testing, Nosetests is used.

On the non-functional side, some tests are missing and some refactoring is clearly needed. At the moment of writing, the code coverage is 70%.

On the function side, I think even more things are missing. Supporting different package managers (now only Conan can be used), different compilers (GCC is used), different versions of C++ (C++ 17 is used), etc.

Check out the issues tab, if that sounds like fun to you.

## Conclusion?

If you're using an IDE that takes care of the generation of a project, of CMakeLists files, good for you, probably you won't need this tool. But if you don't want to use those IDEs, many of them are paying, feel free to try and leave a comment and a star if you liked it.

[Cmake Project Creator](https://github.com/sandordargo/cmake-project-creator) will help you quickly generate new C++ projects based on CMake and Conan dependencies and you can start coding in a matter of seconds if your desired structure is already covered with the built-in options, otherwise in a couple of minutes.

That's clearly a win compared to setting up a new project by hand.