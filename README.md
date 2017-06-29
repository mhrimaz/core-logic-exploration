# core.logic Exploration
Exploration in Clojure core.logic
# Getting Started
Open a Leiningen REPL and for importing core.logic type the following:
```clojure
(require '[clojure.core.logic :as logic])
(require '[clojure.core.logic.pldb :as kb])
(require '[clojure.core.logic.fd :as fd])
```
# Map Coloring
This problem is described in Artificial Intelligence: A Modern Approach book which is a CSP problem.
Here is an image of the map:


![Map Image](https://github.com/mhrimaz/core-logic-exploration/blob/master/MapColoring.PNG)
## Solution
```clojure
(logic/run 4 [q]
           (logic/fresh [WA NT SA Q NSW V T]
             (logic/everyg #(logic/membero % [:RED :GREEN :BLUE]) [WA NT SA Q NSW V T])
             (logic/!= WA NT)(logic/!= WA SA)(logic/!= NT SA)(logic/!= NT Q)
             (logic/!= SA NSW)(logic/!= SA V)(logic/!= Q NSW)(logic/!= NSW V)
             (logic/== q {:WA WA :NT NT :SA SA :Q Q :NSW NSW :V V :T T})
           ))
```


![Map Image 1](https://github.com/mhrimaz/core-logic-exploration/blob/master/MapColoring1.png)


![Map Image 2](https://github.com/mhrimaz/core-logic-exploration/blob/master/MapColoring2.png)

# Simple Finite Domain Puzzle
Find all the numbers between 2 and 13 such x and y which their negation is equal to 7 ( x - y = 7 )
## Solution
```clojure
(logic/run* [q]
            (logic/fresh [x y]
                   (fd/in x y (fd/interval 2 13))
                   (fd/- x y 7)
                   (logic/== q [x y])))
; => ([9 2] [10 3] [11 4] [12 5] [13 6])
```
# Cryptarithmetic Puzzle
This puzzle descirbed in Artificial Intelligence: A Modern Approach page 206. Each letter in a cryptarithmetic puzzle represents a different digit.


![Cryptarithmetic Puzzle](https://github.com/mhrimaz/core-logic-exploration/blob/master/cryptarithmetic.png)


This would be represented as the global constraint Alldifferent (F, T, U, W, R, O)
The addition constraints on the four columns of the puzzle can be written as the following n-ary constraints:
* O + O = R + 10 · C10
* C10 + W + W = U + 10 · C100
* C100 + T + T = O + 10 · C1000
* C1000 = F


Where C10, C100, and C1000 are auxiliary variables representing the digit carried over into the tens, hundreds, or thousands column.

# Solution
```clojure
(defn crypArethmeticPuzzle [q]
  (logic/fresh [T W O F U R TWO FOUR]
               (fd/in T W O F U R (fd/interval 0 9))
               (fd/distinct [T W O F U R]) ; Alldiff ( T W O F U R )
               (fd/in TWO (fd/interval 100 999)) ; TWO is a three digits number
               (fd/in FOUR (fd/interval 1000 9999)) ; FOUR is a four digits number
               
               (fd/eq (= TWO (+ (* 100 T) (* 10 W) O))) ; TWO is consist of three digits which is T W O
               (fd/eq (= FOUR (+ (* 1000 F) (* 100 O) (* 10 U) R))) ; FOUR is consist of four digits which is F O U R
               (fd/eq (= (+ TWO TWO) FOUR)) ; Peace of Cake! TWO + TWO = FOUR
               
               (logic/== q [TWO TWO FOUR]) 
               ))
; => #'user/crypArethmeticPuzzle

(logic/run* [q]
            (crypArethmeticPuzzle q))
; => ([734 734 1468] [765 765 1530] 
; [836 836 1672] [846 846 1692] 
; [867 867 1734] [928 928 1856] [938 938 1876])
```
