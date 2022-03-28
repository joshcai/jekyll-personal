---
layout: post
title:  "Building a 24 Solver in Clojure"
date:   2022-03-27 06:13:34 +0000
categories: tech
shortlink: 24-solver
---

Towards the end of 2019, I was interested in learning a Lisp language - partially because I had never fully taken the time to understand the functional programming paradigm and partially because of Paul Graham's post on [Beating the Averages](http://www.paulgraham.com/avg.html) where he calls Lisp a "secret weapon".

Clojure seemed to be a promising modern Lisp, and I really liked how it could target different languages to compile to, e.g. Java or JavaScript, so I decided to go with it.

To start, I used [4clojure](https://github.com/4clojure/4clojure) (amazing name by the way), which was incredibly helpful to learn the basics and also understand how to do things simpler by looking at other's solutions. It seems that the project has been discontinued but the spirit lives on at [4ever-clojure](https://4clojure.oxal.org/).

Once I felt like I had the basics nailed down, I tried coming up with a project that I could create efficiently using Clojure's strengths - eventually I settled on building a [24 solver](https://joshcai.com/24/).

24 is a pretty simple math game where you have four numbers and your goal is to get the number 24 using any of the standard operations (add, subtract, multiply, divide).

Example:

```
5 7 3 2
```

One solution would be:

```
  2 + 7 + 3 * 5
= 2 + 7 + 15
= 9 + 15
= 24
```

I used to play this with my sister when we were younger - we would just use a deck of playing cards to generate the numbers, and we would race to see who could come up with 24 first. 

To solve this with Clojure, I essentially just brute forced by trying all permutations of numbers with all possible selections of operations. Since you can pass functions around as values, it makes it really simple to do the operation later as part of a reduce step.

Here's the code for the core algorithm:

```clojure
;; Define extra operations for division and subtraction
;; which are not commutative.
(defn- div [a b] (/ b a))
(defn- sub [a b] (- b a))
(defn- generate-permutations [nums]
  ;; Permutations are generated in the following form:
  ;;   [num1 num2 num3 num4] [op1 op2 op3]
  ;; e.g.: 
  ;;   [1 2 8 20] [+ - /]
  ;; When operated on, it would produce
  ;;   (/ 20 (- 8 (+ 2 1)))
  (combo/cartesian-product 
    ;; Generates all permutations of the numbers passed in.
    (combo/permutations nums)
    ;; Generates all possible selections of (n - 1) operations.
    ;; The operations can be the same in the selection.
    (combo/selections [+ - * / div sub] (- (count nums) 1)))
)

(defn- operate [num tup]
  ;; Unpack the next number and operation and perform
  ;; it on the existing number.
  (let [[num2 op] tup]
    (op num2 num)))

(defn- reduce24 [perm]
  (let [[nums ops] (seq perm)]
    (reduce operate
            ;; Grab the first number to start the reduce.
            (first nums)
            ;; Zip the rest of the numbers up with the operations
            ;; to pass to the operate function.
            (map vector (rest nums) ops))))

;; Computes the absolute value of a number.
(defn- abs [n] (max n (- n)))

(defn solve [nums]
  "Solves for 24 given a list of numbers in nums."
  (filter (fn [perm]
    ;; Since ClojureScript does not support ratios, we check that
    ;; the result is within a small epsilon of the target.
    (< (abs (- 24 (reduce24 perm))) 0.00001))
    (generate-permutations nums)
  )
)
```

With that code, you could just call `(solve [2 4 8 5])` to see the list of solutions. Note: it is not in human-friendly notation - printing it in Lisp or PEMDAS notation is actually another challenge in itself. I won't cover that here, but you can check out how I did it in my GitHub repo: [github.com/joshcai/24](https://github.com/joshcai/24).

I was really impressed with how surprisingly simple and concise it was to write this in Clojure. That said, I think I personally would only use Clojure for small projects - it does seem like it could get complicated easily without strong typing. 

What's special about my 24 solver versus all the others? There's some features that I personally wanted that I could never find in any other 24 solver online:

- be able to see if there is a solution without seeing the solutions (not all sets of numbers have solutions)
- generate a random set of 4 numbers to play a new round
- change the target from 24 to another number

Looking for an interesting 24 problem? Try this one:

```
10 4 10 2
```

In my opinion, this one is fiendishly difficult - there is actually only one correct answer, and it's...rather creative. If you get stuck, try out [my solver](https://joshcai.com/24/?a=10&b=4&c=10&d=2&target=24) for this problem.