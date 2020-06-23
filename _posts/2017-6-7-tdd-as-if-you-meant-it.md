---
layout: post
title: "TDD as if you meant it"
date: 2017-6-7
category: dev
tags: [tdd, kata, dojo, cpp, learning]
header: "Recently I had the chance to participate at a coding dojo which was super inspirational.
"
---
The kata we picked was [TDD as if you meant it](http://coderetreat.org/facilitating/activities/tdd-as-if-you-meant-it). The rules are the ones of TDD, plus some additionals. These rules - such as write the code first at the test class and don't move it until... - are not so complicated. However you might end up using quite some time thinking about whether you are playing by the rules...

As I'm sure you observed, [TDD as if you meant it](http://coderetreat.org/facilitating/activities/tdd-as-if-you-meant-it) doesn't give you a specific programming problem to solve. So in fact we had to pick another kata as well. We wanted to choose something simple, something we know. We picked the [Diamond kata](https://github.com/emilybache/DiamondKata).

The first tests seemed pretty lame.

`ASSERT_EQ("A\n", diamond(1));`

The production code simply returned "A".

`ASSERT_EQ(" A \nB B\n A \n", diamond(2));`

I forgot to mention I paired with a highly experienced architect of our company.

So the prodcution code was still dead stupid as that was the least amount of code necessary to past the test.

```
std::string diamond(size_t size) {
    if (size == 1)
        return "A\n";
    if (size == 2)
        return " A \nB B\n A \n";
}
```

As we put on our blue hat for refactoring, he asked me if I see the duplication. The what? Come on, why don't we implement a normal algorithm here? But no. Still the repetition.... Well... We do return twice, but...

I was told to ignore refactoring for a moment, and let's just stupidly sketch up the next test with some hardcoded responses.

Well... Why not...  

`ASSERT_EQ("  A  \n B B \nC   C\n B B \n  A  \n", diamond(3));`

```
std::string diamond(size_t size) {
    if (size == 1)
        return "A\n";
    if (size == 2)
        return " A \nB B\n A \n";
    if (size == 3)
        return "  A  \n B B \nC   C\n B B \n  A  \n";
}
```

Okay, not let's change a bit the outline of this function:

```
std::string diamond(size_t iSize) {
    if (iSize == 1)
        return "A\n";
    if (iSize == 2)
        return " A \n"\
               "B B\n"\
               " A \n";
    if (iSize == 3)
        return "  A  \n"\
               " B B \n"\
               "C   C\n"\
               " B B \n"\
               "  A  \n";
}
```

Do you see it now, Luke? To be frank, I would have already implemented the algorithm... I was trained on [fastest mode code clashes](https://www.codingame.com/multiplayer/clashofcode)... I don't say it is a virtue, but I usually take bigger leaps. This time though let's make some baby steps.

We started to implement functions such as:

```
std::string makeALineSizeOf1() {
    return "A\n"
}

std::string makeALineSizeOf2() {
    return " A \n"
}

std::string makeBLineSizeOf2() {
    return "B B\n"
}
```

So at that time our diamond function would have been something like this:

```
std::string diamond(size_t size) {
    if (size == 1)
        return makeALineSizeOf1();
    if (size == 2)
        return "makeALineSizeOf2() +
               "makeBLineSizeOf2() +
               "makeALineSizeOf2();
    if (size == 3)
        return "makeALineSizeOf3() +
               "makeBLineSizeOf3() +
               "makeCLineSizeOf3() +
               "makeBLineSizeOf3() +
               "makeALineSizeOf3();
}
```

Time to generalize it a bit. But don't move too fast!

```
std::string makeALineSizeOf(size_t size) {
    std::stringstream ss;
    ss << std::string(size - 1, ' ') << 'A' << std::string(size - 1, ' ');
    return ss.toStr();
}

std::string makeBLineSizeOf(size_t size) {
    std::stringstream ss;
    ss << std::string(size - 2, ' ') << 'B' << ' ' << 'B' << std::string(size - 2, ' ');
    return ss.toStr();
}

std::string makeCLineSizeOf(size_t size) {
    std::stringstream ss;
    ss << std::string(size - 3, ' ') << 'C' << '   ' << 'C' << std::string(size - 3, ' ');
    return ss.toStr();
}
```

Then our diamond function looks like:

```
std::string diamond(size_t size) {
    if (size == 1)
        return makeALineSizeOf(1);
    if (size == 2)
        return makeALineSizeOf(2) +
               makeBLineSizeOf(2) +
               makeALineSizeOf(2);
    if (size == 3)
        return makeALineSizeOf(3) +
               makeBLineSizeOf(3) +
               makeCLineSizeOf(3) +
               makeBLineSizeOf(3) +
               makeALineSizeOf(3);
}
```


You start to see how it goes. By the time we reached this point our time was up, we had to return to our offices. So now it is time to finish up the algorithm:

```
std::string makeLineOfCharacterSizeOf(char character, size_t size) {
    std::stringstream ss;
    ss << std::string(size - (character - 'A' + 1), ' ') << character << std::string(1 + 2*int(character - 'B')) << character <<  std::string(size - (character - 'A' + 1), ' ');
    return ss.str();
}
```

Then the diamond is:

```
std::string diamond(size_t size) {
    if (size == 1)
        return makeALineSizeOf(1);
    if (size == 2)
        return makeALineSizeOf(2) +
               makeLineOfCharacterSizeOf('B', 2) +
               makeALineSizeOf(2);
    if (size == 3)
        return makeALineSizeOf(3) +
               makeLineOfCharacterSizeOf('B', 3) +
               makeLineOfCharacterSizeOf('C', 3) +
               makeLineOfCharacterSizeOf('B', 3) +
               makeALineSizeOf(3);
}
```

We still have a problem with 'A'-s. But that's fine, we can have an if in our makeLineOfCharacterSizeOf():

```
std::string makeLineOfCharacterSizeOf(char character, size_t size) {
    std::stringstream ss;
    if (character == 'A') {
        ss << std::string(size - (character - 'A' + 1), ' ') << character << std::string(size - (character - 'A' + 1), ' ');
    } else {
        ss << std::string(size - (character - 'A' + 1), ' ') << character << std::string(1 + 2*int(character - 'B')), ' ') << character <<  std::string(size - (character - 'A' + 1), ' ');
    }
    return ss.str();
}
```
There are some duplications but we'll get back to that later.

Let's go back to diamond which looks like this now:

```
std::string diamond(size_t size) {
    if (size == 1)
        return makeLineOfCharacterSizeOf('A', 1);
    if (size == 2)
        return makeLineOfCharacterSizeOf('A', 2) +
               makeLineOfCharacterSizeOf('B', 2) +
               makeLineOfCharacterSizeOf('A', 2);
    if (size == 3)
        return makeLineOfCharacterSizeOf('A', 3) +
               makeLineOfCharacterSizeOf('B', 3) +
               makeLineOfCharacterSizeOf('C', 3) +
               makeLineOfCharacterSizeOf('B', 3) +
               makeLineOfCharacterSizeOf('A', 3);
}
```

Finish it! If you remember Mortal Kombat...

Add a new failing test case:

`ASSERT_EQ("   A   \n  B B  \n C   C \nD     D\n C   C \n  B B  \n   A   \n", diamond(4));`

If you understand the pattern, you can see that first you have to add some lines starting from A. Then you add the middle line of the diamond which will appear only once. Then you add the lines you already added in the first phase, but now in reverse order.

```
std::string diamond(size_t size) {
    std::stringstream ss;
    for(int i=0; i<size-1; ++i) {
        ss << makeLineOfCharacterSizeOf('A'+i, size);
    }
    ss << makeLineOfCharacterSizeOf('A'+size-1, size);
    for(int i=size-2; i>=0; --i) {
        ss << makeLineOfCharacterSizeOf('A'+i, size);
    }
    return ss.str();
}
```

We are almost done! Let's put on that blue hat again and start refactoring! First get rid off all those stringsteam to string conversions, except for the last one and pass the stringstream around.

A bit simpler now:

```
std::string diamond(size_t size) {
    std::stringstream ss;
    for(int i=0; i<size-1; ++i) {
        makeLineOfCharacterSizeOf('A'+i, size, ss);
    }
    makeLineOfCharacterSizeOf('A'+size-1, size, ss);
    for(int i=size-2; i>=0; --i) {
        makeLineOfCharacterSizeOf('A'+i, size, ss);
    }
    return ss.str();
}

void makeLineOfCharacterSizeOf(char character, size_t size, std::stringstream& ss) {
    if (character == 'A') {
        ss << std::string(size - (character - 'A' + 1), ' ') << character << std::string(size - (character - 'A' + 1), ' ') << "\n";
    } else {
    ss << std::string(size - (character - 'A' + 1), ' ') << character << std::string(1 + 2 * int(character - 'B'), ' ') << character <<  std::string(size - (character - 'A' + 1), ' ') << "\n";
    }
}
```

There are still some duplications though, and makeLineOfCharacterSizeOf is not so readable. So let's improve it!


```
void makeLineOfCharacterSizeOf(char character, size_t size, std::stringstream& ss) {
    ss  << std::string(size - (character - 'A' + 1), ' ');
    if (character == 'A') {
        ss << character;
    } else {
        ss << character << std::string(1 + 2 * int(character - 'B'), ' ') << character;
    }
    ss << std::string(size - (character - 'A' + 1), ' ') << "\n";
}
```

Seems better, right? I think so. Let's move forward and even change some function names.

```
std::string drawSizeOf(size_t size) {
    std::stringstream ss;
    for(int i=0; i<size-1; ++i) {
        addLineOfCharacterSizeOf('A'+i, size, ss);
    }
    addLineOfCharacterSizeOf('A'+size-1, size, ss);
    for(int i=size-2; i>=0; --i) {
        addLineOfCharacterSizeOf('A'+i, size, ss);
    }
    return ss.str();
}

void Diamond::addLineOfCharacterSizeOf(char character, size_t size, std::stringstream& ss) {
    addEdgeSpaces(character, size, ss);
    addCharacter(character, ss);
    if (character != 'A') {
        addMiddleSpaces(character, size, ss);
        addCharacter(character, ss);
    }
    addEdgeSpaces(character, size, ss);
    addNewLine(ss);
}

void Diamond::addCharacter(char character, std::stringstream& ss) {
    ss << character;
}
void Diamond::addEdgeSpaces(char character, size_t size, std::stringstream& ss) {
    ss << std::string(size - (character - 'A' + 1), ' ');
}

void Diamond::addMiddleSpaces(char character, size_t size, std::stringstream& ss) {
    ss << std::string(1 + 2 * int(character - 'B'), ' ');
}

void Diamond::addNewLine(std::stringstream& ss) {
    ss << "\n";
}
```

It's a bit lengthy but it is way much cleaner.