---
title: Euclidean Distance Score
tags: clojure programming-collective-intelligence
---

Recently, I started rereading excellent book [Programming Collective
Intelligence](http://oreilly.com/catalog/9780596529321). I did not
implement any of the examples first time around, so this time i thought,
i implement each one.

Euclidean distance is a method of calculating a score of how similar two
things are. We get a value between 0 and 1, 1 meaning they are
identical 0 meaning they don't have anything in common.

I am using the movie critics example from the book, converted to a
clojure map,

    (def critics 
         {"Lisa Rose" {"Lady in the Water" 2.5 "Snakes on a Plane" 3.5
                       "Just My Luck" 3.0 "Superman Returns" 3.5
                       "You, Me and Dupree" 2.5 "The Night Listener" 3.0}
          "Gene Seymour" {"Lady in the Water" 3.0 "Snakes on a Plane" 3.5
                          "Just My Luck" 1.5  "Superman Returns" 5.0 
                          "The Night Listener" 3.0 "You, Me and Dupree" 3.5}
          "Michael Phillips" {"Lady in the Water" 2.5 "Snakes on a Plane" 3.0
                              "Superman Returns" 3.5  "The Night Listener" 4.0}
          "Claudia Puig" {"Snakes on a Plane" 3.5 "Just My Luck" 3.0
                          "The Night Listener" 4.5 "Superman Returns" 4.0
                          "You, Me and Dupree" 2.5}
          "Mick LaSalle" {"Lady in the Water" 3.0 "Snakes on a Plane" 4.0
                          "Just My Luck" 2.0 "Superman Returns" 3.0
                          "The Night Listener" 3.0 "You, Me and Dupree" 2.0}, 
          "Jack Matthews" {"Lady in the Water" 3.0 "Snakes on a Plane" 4.0
                           "The Night Listener" 3.0 "Superman Returns" 5.0 
                           "You, Me and Dupree" 3.5}
          "Toby" {"Snakes on a Plane" 4.5 "You, Me and Dupree" 1.0
                  "Superman Returns" 4.0}})


To calculate an Euclidean score between two people, first we need to find
what movies they ranked in common, then for each movie, calculate the
difference in ranks and square it, when we sum all squares we a get
similarity score, all that is need to be done is normalize that score so
that it falls between 0 and 1.

![Euclidean n-dimensions](/images/post/euclidean.png)

This is basically Euclidean distance between two points in n-dimensions,
except we don't take the square root of the sum, because it is
computationally expensive and all we are interested is the order of
the distances, order will remain the same whether we take the square
root or not.

    (defn euclidean [person1 person2]
      (let [shared-items (filter person1 (keys person2))
            score (reduce (fn[scr mv]
                            (let [score1 (person1 mv)
                                  score2 (person2 mv)]
                              (+ scr (Math/pow (- score1 score2) 2))))
                          0 shared-items)]
        (if (= (count shared-items) 0)
          0
          (/ 1 (+ 1 score)))))

Now we can calculate a similarity score between two people,

    user=> (euclidean (critics "Lisa Rose") (critics "Gene Seymour"))
    0.14814814814814814

    user=> (euclidean (critics "Lisa Rose") (critics "Lisa Rose"))
    1.0

    user=> (euclidean (critics "Lisa Rose") {})
    0


This allows us to ask the question which critics are similar to Lisa?

    (defn sort-by-similarity [critics critic]
      (sort-by second
               (reduce (fn[h p]
                         (let [name (first p)
                               prefs (second p)
                               similarity (euclidean critic prefs)]
                           (assoc h name similarity) )) {} critics)))

We iterate through the critics map and calculate similarity score for
each person then sort the map using this score,

    user=> (sort-by-similarity critics (critics "Lisa Rose"))
    (["Gene Seymour" 0.14814814814814814] 
     ["Jack Matthews" 0.21052631578947367] 
     ["Toby" 0.2222222222222222] 
     ["Claudia Puig" 0.2857142857142857] 
     ["Mick LaSalle" 0.3333333333333333] 
     ["Michael Phillips" 0.4444444444444444] 
     ["Lisa Rose" 1.0])
