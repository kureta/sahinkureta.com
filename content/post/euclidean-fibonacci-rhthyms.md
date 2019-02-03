---
title: "Euclidean Rhythms and a Familiar Sequence"
date: 2019-01-31T10:35:35-05:00
description: "Euclidean rhythms are the answer to the question 'How can I fit n beats over m quantized pulses as equally spaced as possible?'"
categories: ["Research"]
featuredImage: "/img/2018/09/28/lindenmayer.jpeg"
dropCap: true
displayInMenu: false
displayInList: true
draft: false
---

A recursive definition of Euclidean rhythms and a familiar sequence.<!--more-->

Euclidean rhythms are the answer to the question "How can I fit n beats over m quantized pulses as equally spaced as possible?" When you condense it down to a single sentence it might seem a bit complicated but the idea is very simple.

Suppose we have a time resolution of 8 pulses per second and we want to fit 3 beats in a second. One way to do that would be:

```python
my_rhythm = [1, 1, 0, 1, 0, 0, 0, 0]
```

The above list has 8 elements, representing 8 equal divisions of time. 1s represent where the beats are. This rhythm can also be written as:

```python
my_rhythm = [1, 2, 5]
```

In this representation, numbers in the list represent the time duration of each beat. We will use both of these two representations and I will refer to them as "pulse" and "beat" representations respectively. And a note: Euclidean rhythms are in fact cycles, not lists with a beginning and an end. Therefore `[1, 2, 5]` is equal to `[2, 5, 1]` and `[5, 1, 2]`. Think of these not as numbers on a list but numbers around a circle, and rotating the circle doesn't change anythin. There are no first or last points on a circle.

Musical notation of (on instance of) this beat pattern is like this:

```python
import music21
music21.converter.parse("tinynotation: c8 c4 c8~ c2").show()
```

![](/img/2018/09/28/Fibonacci Rhythms_6_0.png)

Let's get back to our initial question. Does the above rhythm look "as equally spaced as possible". Not at all. The first beat is only a single pulse long and the last beat is a whopping 5 pulses long. We can definitely do better. For example, we can take pulse from the longest beat and give it to the shortest one.

```python
my_rhythm = [2, 2, 4]
```

This looks more like it but I think we can do better.

```python
my_rhythm = [3, 2, 3]
```

Or in notation:

```python
music21.converter.parse("tinynotation: c4. c4 c4.").show()
```

![](/img/2018/09/28/Fibonacci Rhythms_12_0.png)

Can we do any better then? Suppose we give one pulse from one of the 3-pulse long beats to the 2-pulse long one.

```python
my_rhythm = [3, 3, 2]
```

It feels like we did not change much. Remember, Eucidean rhythms are actually cycles. So `[3, 2, 3]` is equal to `[3, 3, 2]`. So this is as equally spread as we can get given 8 pulses and 3 beats. Turns out Euclidean rhythms can be seen in lots of different types of music. You can just Google "Euclidean rhythms" or check out Godfried Toussaint’s paper, ["The Euclidean Algorithm Generates Traditional Musical Rhythms"](http://cgm.cs.mcgill.ca/\~godfried/publications/banff.pdf). If you are interested in more about the mathematical side of this you can search for Björklund's 2003 paper "The Theory of Rep-Rate Pattern Generation in the SNS Timing System". He is the one who discovered the algorith for generating Euclidean rhythms. By the what they are called that because the algorithm used to generate these rhythms is almost the same as Euclid's algorithm for finding the [greatest common divisor](https://en.wikipedia.org/wiki/Greatest_common_divisor) of two integers.

So, how can we find the euclidean rhythm that corresponds to 3 beats in 8 pulses. We need a function `euclid(number_of_beats, number_of_pulses)` that will give us an euclidean rhythm given the number of beats and quantization of time. If we had 6 pulses it would have been very easy, as 6 can be divided into three, and we would have this beat pattern: `[2, 2, 2]`, which is actually equivalent to `[2]` since these rhythms are actually cycles, but that is not important right now. Anyways, since we are trying to divide 8 pulses into 3 beats, we still have a left over duration of 2 pulses. Now we have to find a way to equally spread this remaining 2 pulses over 3 beats. So, in order to solve `euclid(3, 8)`, we need to solve `euclid(2, 3)`. Let's cheat a bit. I will just tell you what `euclid(2, 3)` is. It is `[2, 1]` in beat representation, or `[1, 0, 1]` in pulse representation. It's pulse representation tells us how to evenly spread the remaining 2 pulses over `[2, 2, 2]` so that the resulting rhythm is 8 pulses long and as evenly spread as possible. Add the elements of the equal divisions beat representation to remaining Eculidean rhythms pulse representation. And the result is `[3, 2, 3]`.

Euclidean rhythm for 2 beats and 8 pulses is simply 8 / 2 = 4. Two beats of 4-pulses each `[4, 4]` or equivalently just `[4]`.

Now we can come up with a recursive definition for our `euclid(num_beats, num_pulses)`

    if number of beats equally divides number of pulses:
        Euclidean rhythm is just a single beat and its duration is the result of that division.
    else:
        Euclidean rhythm is the remainder of (number of pulses / number of beats) equally spread over the quotient of the same division.

Let's do this!

```python
from itertools import cycle
```

```python
def pulses_from_beats(beats):
    """This function converts a beat representation into a pulse representation."""
    return sum([[1] + [0] * (i - 1) for i in beats], [])

class Rhythm:
    """We define a simple Rhythm class in order to avoid calling the above function everytime we need a rhythm's pulse representation."""
    def __init__(self, beats):
        self.beats = beats
        self.pulses = pulses_from_beats(beats)

    def __repr__(self):
        return self.beats.__repr__()
```

```python
def spread_over_equal_division(rhythm, equal_div):
    """Given an Euclidean rhytm, spreading it over an equal division is just adding its pulse representation
    to the equal division's beat representation."""
    return Rhythm([p + b for p, b in zip(rhythm.pulses, cycle([equal_div]))])
```

```python
def euclid(num_pulses, num_beats):
    """divmod function gives us both the quotient and the remainder of an integer division."""
    quotient, remainder = divmod(num_pulses, num_beats)
    if remainder == 0:
        # If beats equally divides the pulses duration all beats are the same duration.
        return Rhythm([quotient])
    else:
        # Else we spread the euclidean rhythm generated from the remainder, over the equally dividing part.
        return spread_over_equal_division(euclid(num_beats, remainder), quotient)
```

Let's try to generate some Euclidean rhythms!

```python
euclid(8, 5)
```

    [2, 1, 2, 2, 1]

In musical notation:

```python
music21.converter.parse("tinynotation: c4 c8 c4 c4 c8").show()
```

![](/img/2018/09/28/Fibonacci Rhythms_26_0.png)

Now, apart from being able to generate Euclidean rhythms from a pair of integers, number of beats and number of pulses, we have another toy to play with; our `spread_over_equal_division(rhythm, equal_div)` function. First thing that came to my mind when I implemented this recursive version of the Bjorklund's algorithm was to generate Euclidean rhythms from the simplest building blocks in terms of this function. The simplest Euclidean rhythm, which is also an equal division, is `[1]`. Not very exciting. But we can generate infintely many Euclidean rhythms using just `[1]` as our seed, and the `spread_over_equal_division(rhythm, equal_div)` function, like this:

```python
next_rhythm = spread_over_equal_division(Rhythm([1]), 1)
next_rhythm
```

    [2]

Still nothing too exciting. Just an equal division. Let's go on anyway. Spread this rhythm over 1.

```python
next_rhythm = spread_over_equal_division(next_rhythm, 1)
next_rhythm
```

    [2, 1]

Slightly more exiting. At least has some variation. Let's do that again.

```python
next_rhythm = spread_over_equal_division(next_rhythm, 1)
next_rhythm
```

    [2, 1, 2]

And again...

```python
next_rhythm = spread_over_equal_division(next_rhythm, 1)
next_rhythm
```

    [2, 1, 2, 2, 1]

One last time...

```python
next_rhythm = spread_over_equal_division(next_rhythm, 1)
next_rhythm
```

    [2, 1, 2, 2, 1, 2, 1, 2]

There are a couple of interesting things going on. Let me list our rhythms for convenience, don't forget, we have started with `Rhythm([1])`.

    [1]
    [2]
    [2, 1]
    [2, 1, 2]
    [2, 1, 2, 2, 1]
    [2, 1, 2, 2, 1, 2, 1, 2]

First off, notice every entry is the concatenation of previous two entries. Hmmm... This might remind some of you a certain sequence generated via a smilar process.

Secondly, these rhythms are equivalent to:

    euclid(1, 1)
    euclid(1, 2)
    euclid(2, 3)
    euclid(3, 5)
    euclid(5, 8)
    euclid(8, 13)

Yes, these are pairs of consecutive Fibonacci numbers! We have found an analog of the Fibonacci sequence within Euclidean rhythms. Since the limit of the ratio of two consecutive Fibonacci numbers approach the golden ratio, which is "maximally irrational" in a certain sense that is related to its continued fraction representation, these rhythms are also "maximally interesting" in a certain sense. The sequence can be extended infinitely and it will never repeat itself. It is also closely related to L-Systems. In fact it is identical to the first "deterministic, contentext-free L-System (DOL-System) in Lindenmayer's Book, [The Algorithmic Beauty of Plants](http://algorithmicbotany.org/papers/#abop). Just replace **a**s with **2**s and **b**s with **1**s.

Interestingly, the way he generates the same sequence has nothing to do with Bjorklund's or Euclid's algorithm, Greatest Common Divisors, or concatanation of strings. The rule he uses to generate the same sequence, translated to **1**s and **2**s, is:

    2 -> 21
    1 -> 2

Starting with 1 and replacing everything according to above rules yields:

    1
    2
    21
    212
    21221
    21221212

Identical to our own fibonacci sequence.

_- Sahin Kureta (20180929)_
