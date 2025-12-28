+++
title = 'Advent Of Code 2025'
date = 2025-12-05
description = 'My Advent of Code 2025 solutions in Clojure.'
tags = ['Clojure']
+++

The [Advent of code](https://adventofcode.com/2025) started a few days ago, and this year I decided to implement my solutions in Clojure.

My solutions in this repository: https://github.com/dimiro1/adventofcode2025-clj

## Day 11

Source: https://github.com/dimiro1/adventofcode2025-clj/blob/main/src/day11.clj

### Part 1

Again, part 1 was easy! You can implement a naive [DFS](https://www.geeksforgeeks.org/dsa/number-of-paths-from-source-to-destination-in-a-directed-acyclic-graph/) to walk on the tree. It's slow, but it worked for part 1.

### Part 2

Owwww, definitely walking down the tree using a simple depth search won't work here. I had to revisit some tree navigation algorithms in order to find a solution. The solution here is to avoid recomputations with a technique called memoization (I think they call this dynamic programming).

## Day 10

Source: https://github.com/dimiro1/adventofcode2025-clj/blob/main/src/day10.clj

### Part 1

As usual, part 1 is not difficult. Yes, it takes some time, but it's not difficult. My solution here was to try different combinations growing in size, first I tried each individual box, then a combination of 2, 3, and so on, until I found the solution.

Clojure shined on this problem; using [some](https://clojuredocs.org/clojure.core/some) and [reduce](https://clojuredocs.org/clojure.core/reduce) was lovely.

### Part 2

Tooooo hard... I'm not implementing it.

## Day 9

Source: https://github.com/dimiro1/adventofcode2025-clj/blob/main/src/day9.clj

### Part 1

This one was a good surprise. I woke up early and jumped into the problem, and then after a few minutes I had the solution.

It is a trivial problem; you just need to compute the area of all combinations and then choose the largest one.

### Part 2

Hahahaha, not funny at all. At some point I came up with a solution of computing a polygon and then finding only the boxes that fell into the polygon. This was very hard. I don't know, it did not work, then I just gave up after some time.

## Day 8

Source: https://github.com/dimiro1/adventofcode2025-clj/blob/main/src/day8.clj

### Part 1

Not difficult. I just needed to compute the [Euclidean distance](https://en.wikipedia.org/wiki/Euclidean_distance) for each box and then keep connecting them together.

### Part 2

I was not able to solve part 2; this one was hard.

## Day 7

Source: https://github.com/dimiro1/adventofcode2025-clj/blob/main/src/day7.clj

### Part 1

This one was fun. My solution was to simulate what happens on each row, in the end recreating the final matrix and then just counting how many times I have the pattern `|ˆ|`.

### Part 2

This was the first day that I was not able to solve the problem. I think this is due to how I implemented my solution for part 1. If you don't have a good initial solution that you can expand to the second one, for sure there will be issues on the second part, which is usually about optimizations.

## Day 6

Source: https://github.com/dimiro1/adventofcode2025-clj/blob/main/src/day6.clj

### Part 1

Trivial. Parsing each number and then applying the function `+` or `*` on the numbers, jumping every n (length of the row) times.

e.g.

```markdown
123 328  51 64
 45 64  387 23
  6 98  215 314
*   +   *   +
```

Length is 4, so `(* 123 45 6)`.

### Part 2

This one took me so much time to figure out. I had to sleep on it to find the solution.

Taking from the example above, I had to find where the numbers are divided, and for that I used the last row. Before each operation I added a mark `X`, which then tells me how to split the numbers above. With this in mind, and to simplify my life, I added zeros in the remaining empty positions. This was needed so that `60` can become `600`. The good thing is that this will not affect the left side: `060` becomes `60`, so all good.

The next preparation was to rotate the numbers, so that:

```clojure
;;; [123 45 6] -> [356 24 1]
```

After this preparation, I just needed to perform the math as done in part 1.

## Day 5

Source: https://github.com/dimiro1/adventofcode2025-clj/blob/main/src/day5.clj

### Part 1

I just needed to check if the numbers are in the ranges. It was quite trivial to implement.

### Part 2

My first attempt was to use the same logic from part 1, but it failed miserably. The ranges are too large to expand.

My final solution was to merge the overlapping ranges. This solves 95% of the challenge; the rest is just reducing into a number.

## Day 4

Source: https://github.com/dimiro1/adventofcode2025-clj/blob/main/src/day4.clj

### Part 1

This was a nice one, nothing fancy. After implementing a function to get a value from any direction, the problem became just applying a simple function to filter out the rolls.

### Part 2

Similar to part 1, but in this part I had to keep "mutating" the board to apply the logic again.

## Day 3

Source: https://github.com/dimiro1/adventofcode2025-clj/blob/main/src/day3.clj

### Part 1

I walked the battery bank from each side and tried to find the largest one from the left and from the right.

### Part 2

For part 2, the idea was to implement a [backtracking algorithm](https://www.geeksforgeeks.org/dsa/backtracking-algorithms/). It was not trivial to figure out. I had to read a few algorithm pages on Wikipedia and from https://www.geeksforgeeks.org to fully grasp the implementation.

## Day 2

Source: https://github.com/dimiro1/adventofcode2025-clj/blob/main/src/day2.clj

### Part 1

The implementation is simple: expand the ranges, then split the number into two parts, then just check if the parts are the same.

### Part 2

For part 2, the algorithm from part 1 does not fit. The numbers on the edges of the ranges are too large. My initial idea was to avoid expanding the ranges, but I did try anyway, and it worked, even though it takes a couple of minutes to run.

For each number I had to incrementally split it one at a time for different split increments: 2, 3, 4, and so on. If in the end I still have only one part, then the number is valid.

## Day 1

Source: https://github.com/dimiro1/adventofcode2025-clj/blob/main/src/day1.clj

### Part 1

This one was trivial. The idea was to use the [mod](https://clojuredocs.org/clojure.core/mod) function to simulate the rotation.

### Part 2

Part 2 was harder. Initially I could not find the math to check when the rotation crosses 0. In the end, after spending a couple of hours, I gave up and implemented a brute force solution where I simulate each and every movement on the knob. Not elegant, but it works.
