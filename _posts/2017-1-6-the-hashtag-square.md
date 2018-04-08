---
layout: post
title: "The Hashtag Square"
date: 2017-1-6
category: dev
tags: [c#, school, code]
header: "My brother is learning programming in high school and from time to time I give him some exercises. Due to various reasons often we don't use video calls to discuss, but I rather write down my explanations.
"
---

So did I recently this week when I wanted to explain to him and his classmate how to implement the below exercise. So why not to publish it in English on my new blog?!

The goal is to take an input from the user and draw an empty square bordered by hashtags on the standard output.

Let's see a few examples, if you prefer criteria for your acceptance tests:

```
Input: 
1

Output:
#
```

```
Input: 
2

Output:
##
##
```

```
Input: 
3

Output:
###
# #
###
```

```
Input: 
4

Output:
####
#  #
#  #
####
```

```
Input: 
5

Output:
#####
#   #
#   #
#####
```

Before starting the implementation what can we see? 
- Each line of a square has the same width.
- The first and the last lines contain as many hashtags as your input indicates.
- The middle lines have only two hashtags. One on the left side and one on the right. In between you have input minus two spaces. Two hashtags and input minus two spaces make exactly input number of characters, so the width is still fine.
- We have total lines (which is equal to width) minus two middle lines. It's meant to be a square in the end, so no surprise.


Let's see some pseudocode first:

```
width = input()

firstLine = ""
for (i=0, i<width, i++) {
    firstLine += "#"
}
print firstLine


for (i=0; i<width-2; i++) {
    middleLine = "#"
    for (j=0; j<width-2; j++) {
        middleLine += " "
    }
    middleLine += "#"
    print(middleLine)
}

lastLine = ""
for (i=0, i<width, i++) {
    lastLine += "#"
}
print lastLine
```

Hell yeah, this is full of duplication, but don't shout this is just a pseudo code to make you understand things. Are we fine? For the width of 4 we are. Our script will print four hashtags in the first line, then it will print two lines where we have two hashtags in the middle and finally we repeat the first line.

Let's check the case of width 2. The critical point is printing the middle lines. `width - 2 = 0` which is not greater than `i`'s initial value of zero, so no middle line will be printed! That is just awesome!

What about the case of input 1? We print one hashtag as first line, than no middle lines and then... Oh no! We print a last line! But we shouldn't as the first and the last lines should be the same for the size of one. So we need one more condition if we want to accept 1 as an input. Either check the input in the beginning or just before attempting to print the last line. The previous one needs one line less, the latter is more readable to me. I choose the latter one, but it's up to you.

```
width = input()

if (width == 1) {
    print('#')
    return
} 
firstLine = ""
for (i=0, i<width, i++) {
    firstLine += "#"
}
print firstLine


for (i=0; i<width-2; i++) {
    middleLine = "#"
    for (j=0; j<width-2; j++) {
        middleLine += " "
    }
    middleLine += "#"
    print(middleLine)
}

lastLine = ""
for (i=0, i<width, i++) {
    lastLine += "#"
}
print lastLine
```

I don't speak C#, but as I had to provide an implementation in that language, so maybe it is not following all the conventions. Here is a version where I implemented the pseudocode keeping all its duplications.

```
using System;

public class Program
{
    public static void Main(string[] args)
    {
        int width = Convert.ToInt32(Console.ReadLine());
        if (width == 1) {
            Console.WriteLine("#");
            return;
        }
        
        String firstLine = "";
        for (int i=0; i<width; i++) {
            firstLine += "#";
        }
        Console.WriteLine(firstLine);
        
        for (int i=0; i<width-2; i++) {
            String middleLine = "#";
            for (int j=0; j<width-2; j++) {
                middleLine += " "; 
            }
            middleLine += "#"; 
            Console.WriteLine(middleLine);
        }

        
        String lastLine = "";
        for (int i=0; i<width; i++) {
            lastLine += "#";
        }
        Console.WriteLine(lastLine);
    }
}
```

Here is a next version, where I created some functions in order to enhance readability and to remove some of the code duplication:

```
using System;

public class Program
{
    public static String makeOuterLine(int width) {
        String outerLine = "";
        for (int i=0; i<width; i++) {
            outerLine += "#";
        }
        return outerLine;
        
    }
    
    public static String makeInnerLine(int width) {
        String innerLine = "#";
        for (int i=0; i<width-2; i++) {
            innerLine += " "; 
        }
        innerLine += "#"; 
        return innerLine;
        
    }
    
    public static void drawSquareOfOne() {
        Console.WriteLine(makeOuterLine(1));
    }
    
    public static void drawSquareOfWidth(int width) {
        Console.WriteLine(makeOuterLine(width));
        
        for (int i=0; i<width-2; i++) {
            Console.WriteLine(makeInnerLine(width));
        }
        
        Console.WriteLine(makeOuterLine(width));
    }
    
    public static void Main(string[] args)
    {
        int width = Convert.ToInt32(Console.ReadLine());
        if (width == 1) {
            drawSquareOfOne();
        } else {
            drawSquareOfWidth(width);
        }
    }
}
```

Still we have some unnecessary for loops as most of the languages somehow will let you multilply characters or even strings. So here is the final version:

```
using System;

public class Program
{
    static char BUILDING_BLOCK = '#';
    static char EMPTY_BLOCK = ' ';
    
    public static void Main(string[] args)
    {
        int width = Convert.ToInt32(Console.ReadLine());
        if (width == 1) {
            drawSquareOfOne();
        } else {
            drawSquareOfWidth(width);
        }
    }
    
    public static void drawSquareOfOne() {
        Console.WriteLine(makeOuterLine(1));
    }
    
    public static void drawSquareOfWidth(int width) {
        Console.WriteLine(makeOuterLine(width));
        
        for (int i=0; i<width-2; i++) {
            Console.WriteLine(makeInnerLine(width));
        }
        
        Console.WriteLine(makeOuterLine(width));
    }

    public static String makeOuterLine(int width) {
        return new String(BUILDING_BLOCK, width);
    }
    
    public static String makeInnerLine(int width) {
        return BUILDING_BLOCK 
        + new String(EMPTY_BLOCK, width - 2) 
        + BUILDING_BLOCK;
    }
}

```

I hope you recognize that variable names and method organization also evolved between the different versions. God bless Uncle Bob for his Clean Code videos!
