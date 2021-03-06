---
title: Yet Another Brute Force Sudoku Solver
tags: clojure
---

I know, there are more than a thousand sudoku solvers out there, but
I've been meaning to learn the puzzle for a while now, on the other hand
I have no interest in solving one for real so I hacked together the
following snippet to brute force it for me.

     (ns sudoku.core
       (:use clojure.set)
       (:use clojure.contrib.seq-utils))

     (defn constraints [s i]
       (let [s (partition 9 s) r (/ i 9) c (mod i 9) gc (/ c 3) gr (/ r 3)
             every-nth (fn [s i] (map #(nth % i) s))
             grp-col (every-nth (map #(partition 3 %) s) gc)
             grp (take 3 (drop (* 3 (int gr)) grp-col))]
         (into #{} (flatten [(nth s r) (every-nth s c) grp]))))

Rules are simple, when choosing a number, you can't use numbers that are
already present in the same column, same row, and the same group, the
set of these numbers will contain invalid choices for that position.

     (defn solve [s]
       (if (.contains s 0)
         (let [i (.indexOf s 0)
               inject #(concat (take %2 %1) [%3] (drop (inc %2) %1))]
           (flatten (map #(solve (inject s i %))
                         (difference #{1 2 3 4 5 6 7 8 9} (constraints s i))))) 
         s))

For every empty position in the puzzle, we calculate the set of invalid
choices, difference between the invalid choices and the set of numbers
from 1 to 9 gives us the possible choices for that location, we inject
each possible choice into this location, call solve on these new
sudokus until a solution is found.

     (solve [3 0 0 0 0 5 0 1 0
             0 7 0 0 0 6 0 3 0
             1 0 0 0 9 0 0 0 0
             7 0 8 0 0 0 0 9 0
             9 0 0 4 0 8 0 0 2
             0 6 0 0 0 0 5 0 1
             0 0 0 0 4 0 0 0 6
             0 4 0 7 0 0 0 2 0
             0 2 0 6 0 0 0 0 3])

     (3 8 6 2 7 5 4 1 9 
      4 7 9 8 1 6 2 3 5 
      1 5 2 3 9 4 8 6 7 
      7 3 8 5 2 1 6 9 4 
      9 1 5 4 6 8 3 7 2 
      2 6 4 9 3 7 5 8 1 
      8 9 3 1 4 2 7 5 6 
      6 4 1 7 5 3 9 2 8 
      5 2 7 6 8 9 1 4 3)

     (solve [5 3 0 0 7 0 0 0 0
             6 0 0 1 9 5 0 0 0
             0 9 8 0 0 0 0 6 0
             8 0 0 0 6 0 0 0 3
             4 0 0 8 0 3 0 0 1
             7 0 0 0 2 0 0 0 6
             0 6 0 0 0 0 2 8 0
             0 0 0 4 1 9 0 0 5
             0 0 0 0 8 0 0 7 9])

     (5 3 4 6 7 8 9 1 2
      6 7 2 1 9 5 3 4 8
      1 9 8 3 4 2 5 6 7
      8 5 9 7 6 1 4 2 3
      4 2 6 8 5 3 7 9 1
      7 1 3 9 2 4 8 5 6
      9 6 1 5 3 7 2 8 4
      2 8 7 4 1 9 6 3 5
      3 4 5 2 8 6 1 7 9)
