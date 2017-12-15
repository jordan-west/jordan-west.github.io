---
layout: post
title: Working with Bits 
comments: true
tags: tutorial c++
---

I've been taking part in an event called the [Advent of Code](http://adventofcode.com), where you are given a programming problem to solve each day for the first 25 days of December. In this post I want to look at [Day 15](http://adventofcode.com/2017/day/15) as it requires knowing how to work with the underlying bits of a number.

<!--more-->

## The problem

The [Advent of Code problem](http://adventofcode.com/2017/day/15) requires performing multiplications and divisions on large integers and then comparing the lowest 16 bits of each pair of results. What does it mean to compare the lowest 16 bits of an integer, and how do we go about doing this in C++?

Say we have two integers that we want to compare, `245556042` and `1431495498`. If we convert these to binary, we get the following:

```c++
// 245556042 in binary:
00001110101000101110001101001010

// 1431495498 in binary:
01010101010100101110001101001010
```

We can see that these are 32 bit values, but we only care about the lowest 16 bits:

```c++
/*
 * The rightmost 16 bits are the lowest and we can
 * see that they are equal in this example.
 */
0000111010100010 [1110001101001010]
0101010101010010 [1110001101001010]
```

## Bitwise operators

To understand how to select certain bits, we need to look at what the bitwise operators do. These operators work on each bit of a number. There are six bitwise operators in C/C++:

```c++
|  // BITWISE OR
&  // BITWISE AND
^  // BITWISE XOR
~  // BITWISE NOT
>> // LOGICAL RIGHT SHIFT
<< // LOGICAL LEFT SHIFT
```

### Bitwise OR operator

The bitwise OR operator results in a `0` if both bits are `0`, and results in a `1` otherwise:

```c++
int x = 5;     // x = 0101
int y = 12;    // y = 1100
int z = x | y; // z = 1101
```

The bitwise OR operator is very useful when you want to set certain states and store the combination of set states in a single value. States that can be set in combination with one another are known as flags, and many libraries take advantage of the bitwise OR operator for working with flags. An example from the [SDL](https://www.libsdl.org) library is shown below:

```c++
// See how each flag is selected in the last parameter:
SDL_Window* window = SDL_CreateWindow("Window", 0, 0, 640, 480, SDL_WINDOW_OPENGL | SDL_WINDOW_RESIZABLE | SDL_WINDOW_HIDDEN);

// Each flag is associated with a distinct bit pattern such as:
// (We set bit patterns with the 0b prefix in C++14)
enum SDL_WindowFlags
{
    SDL_WINDOW_OPENGL     = 0b0001;
    SDL_WINDOW_RESIZABLE  = 0b0010;
    SDL_WINDOW_HIDDEN     = 0b0100;
    SDL_WINDOW_BORDERLESS = 0b1000;
};

// FLAGS_RESULT = 0b0111
int FLAGS_RESULT = SDL_WINDOW_OPENGL | SDL_WINDOW_RESIZABLE | SDL_WINDOW_HIDDEN;
```

In the example above, we can tell that the `SDL_WINDOW_BORDERLESS` flag is not set because the leftmost bit is not set in `FLAGS_RESULT`. But we know that the other three flags are set by looking at their corresponding bits. In the next section, we'll see how we can check whether bits are set.

### Bitwise AND operator

The bitwise AND operator results in a `1` if both bits are `1`, and results in a `0` otherwise:

```c++
int x = 5;     // x = 0101
int y = 12;    // y = 1100
int z = x & y; // z = 0100
```

The bitwise AND operator is very useful for checking whether particular bits are set in a pattern. Continuing our SDL example from above: 

```c++
// Check whether the RESIZABLE flag is set:
if (FLAGS_RESULT & SDL_WINDOW_RESIZABLE == SDL_WINDOW_RESIZABLE)
{
}

// In C++ expressions are true if they have a non-zero value.
// This expression is only non-zero when the correct bit is set.
// Therefore we can shorthand this to:
if (FLAGS_RESULT & SDL_WINDOW_RESIZABLE)
{
}

/*
 * FLAGS_RESULT         = 0111
 * SDL_WINDOW_RESIZABLE = 0010
 * ANDed                = 0010
 */
```

An interesting application of the bitwise AND operator is examining whether a number is odd by checking if the rightmost bit is set. The reason that a number is only odd if the rightmost bit is set is because the rightmost bit is the only power of 2 that is not even.

### Bitwise XOR operator

The bitwise XOR (exclusive OR) operator results in a `0` if both bits are the same, and results in a `1` otherwise:

```c++
int x = 5;     // x = 0101
int y = 12;    // y = 1100
int z = x ^ y; // z = 1001
```

The bitwise XOR operator is useful when you need to toggle a bit on or off:

```c++
int FLAGS_RESULT = SDL_WINDOW_OPENGL | SDL_WINDOW_HIDDEN;

// Unsets the OPENGL flag
FLAGS_RESULT = FLAGS_RESULT ^ SDL_WINDOW_OPENGL;

/*
 * FLAGS_RESULT      = 0101
 * SDL_WINDOW_OPENGL = 0001
 * XORed             = 0100
 */

// Sets the RESIZABLE flag
FLAGS_RESULT = FLAGS_RESULT ^ SDL_WINDOW_RESIZABLE;

/*
 * FLAGS_RESULT         = 0100
 * SDL_WINDOW_RESIZABLE = 0010
 * XORed                = 0110
 */
```

### Bitwise NOT operator

The bitwise NOT operator flips bit values from `0` to `1`, and from `1` to `0`:

```c++
int x = 5;  // x = 0101
int z = ~x; // z = 1010
```

### Logical left shift operator

The left shift operator shifts all bits in a number to the left by some amount:

```c++
int x = 2;     // x = 0010 = 2
int y = x << 1 // y = 0100 = 4
int z = x << 2 // z = 1000 = 8
```

If you examine the values above closely you can see that for each shift to the left the number is multiplied by 2. A multiplication by 2 is not guaranteed because logical shifts do not preserve sign bits or distinguish between a number's exponent and mantissa. Bits may be lost in a logical left shift if they exceed the leftmost bit.

The left shift operator is a useful tool for manipulating individual bits in a number:

```c++
int x = 1 << 2;

/*
 * 1      = 0001
 * 1 << 2 = 0100
 */

// Check if the leftmost bit is set in x:
// (The shift operator takes precedence, so we shift then AND)
if (x & 1 << 3)
{
}

/*
 * x      = 0100
 * 1 << 3 = 1000
 * ANDed  = 0000
 */
```

### Logical right shift operator

The right shift operator shifts all bits in a number to the right by some amount:

```c++
int x = 4;      // x = 0100 = 4
int y = x >> 1; // y = 0010 = 2
int z = x >> 2; // z = 0001 = 1
```

If you examine the values above closely you can see that for each shift to the right the number is divided by 2. For the same reasons stated above in the left shift operator description, a division by 2 cannot be relied on for the logical right shift operator.

## Back to the problem

Now we know how the bitwise operators work, we can return to our problem of checking whether the lowest 16 bits of our pair of numbers are equal. We know that the bitwise AND operator can be used to check if bits are set, and we want to check the lowest 16 bits so:

```c++
int x = 245556042;
int y = 1431495498;

// Check if lowest 16 bits are equal:
if ((x & 0xffff) == (y & 0xffff))
{
}

/*
 * 0xffff is the hexadecimal for:
 * 00000000000000001111111111111111
 * f is the hexadecimal for 15 which is 1111 in binary.
 */
```

Although the solution to the problem only required one of the six bitwise operators, hopefully this post helps in building confidence to work with bits in a number of different ways.
