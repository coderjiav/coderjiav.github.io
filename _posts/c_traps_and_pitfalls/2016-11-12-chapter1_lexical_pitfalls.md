---
layout: post
title:  "Chapter 1 Lexical Pitfalls"
date:   2016-11-12 15:45
location: szu library
tag:  
- C
- traps and pitfalls
categories: C Traps and Pitfalls
---
> **The same sequence of characters can belong to one token in one context and an entirely different token in another context.**

> The part of a compiler that breaks a program up into tokens is often called a lexcial analyzer

# = is not ==
C use = for assignment and == for comparison. Moreover, C treats = as an operator, so that multiple assignments( such as a=b=c ) can be written easily and assignments can be embedded in larger expressions.

This convenience causes a potential problem: one can inadventently write an assignment where intended a comparison. Refer below code.

```cpp
    if (x =y)
        break;  
```
This code intended to check if x is equal to y, but now the behavior of this expression is different. It assign y's value to x, then check if x is equal to zero. The break will excute whenever y is not zero. 

Consider the following loop, which intended to skip blanks, tabs and newlines in a file

```cpp
    while (c = ' ' || c == '\t' || c == '\n')
        c = getc(f);
```
Notice the first = , it should be ==; Because the = operator has lower precedence than the || operator, the 'comparison' actually assigns to c the value of entire expression:


```cpp
    ' ' || c == '\t' || c == '\n'
```
Since this is a boolean value and ' ' is not zero, so the test value is always true; this maybe cause the loop runing forever.

Consider following example which we should use = instand of ==:


```cpp
    if ( (  filedesc == open(argv[i], 0) ) < 0) {
        error();
    }
```
It intended to assign the result of file open to filedesc, then compare it with zero, but instand of assignment, it compares the result with filedesc. The result of comparison is always 0 or 1,so the error will not call forever.

Some compiler might warn that the comparison with 0 has no effect, but you shouldn't count on it.

  
#  & and | are not && or ||

# Greedy lexical analysis
When a C compiler encounters a / followed by an *, it must be able to decide whether to treat these two characters as two separate tokens or as one single token.

C resolves this question with a simple rule: repeatedly bite off the biggest possible piece.That is, the way to convert a C program to tokens is to move from left to right, taking the longest possible token each time. This strategy is also sometimes referred to as greedy, or, more colloquially, as the maximal munch strategy.Tokens never contain embedded white space.

Eg.  == is a single token, = = is two.

     a--b means the same as a-- -b because gready strategy. -- is a operator. 

Another example:
    y = x /*p  /* p point at the divisor */ ;

    Since compiler encounter / and * next to it , it will treat /* as one token, so this expression just assign x to y;
# Integer constants
# Strings and charactes
Single and double quotes mean very different things in C, and confusing them in some contexts will result in surprises rather than error messages.

> A character enclosed in single quotes is just another way of writing the integer that corresponds to the given character in the implementation's collating sequence. Thus, in an ASCII implementation, 'a' means exactly the same thingg as 0141 or 97.

> A string enclosed in double quotes, on the other hand, is a short-hand way of writing a pointer to the initial character of a nameless array that has been initialized with the characters between the quotes and an extra character whose binary value is zero.

```cpp
    printf("Hello, World\n");
    is equivalent to
    char hello[] = {'H', 'e', 'l', 'l', 'o', ',' ,'W', 'o', 'r', 'l', 'd', '\n', 0}
```
Because a character in single quote represents an integer and a character in double quotes represents a pointer, compiler type checking will ususlly catch places where one is used for the other. Thus, the following code will yield an error message because '/' is not a character pointer.


```cpp
    char *slash = '/';
```

However, some implementations don't check argument types, particularly arguments to printf.Thus saying


```cpp
    printf('\n');
        instead of 
    printf("\n");
```
may result in a surprise at run time instead of a compiler diagnostic.

Because an integer is usually large enough to hold several characters, some C compilers permit multiple characters in a character constant as well as a string constant.That means that writing 'yes' instand of "yes" may well go undetected. 'yes' is not precisely defined, but many C implementations take it to mean "an integer that is composed somehow of the values of the characters y, e and s".    