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

## Solution
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
# Sudoku
The goal of Sudoku is to fill a 9×9 grid with numbers so that each row, column and 3×3 section contain all of the digits between 1 and 9. The constraints are:
* Each column must contain one occurrence of each digit from 1–9.
* Each row must contain one occurrence of each digit from 1–9.
* Each 3 x 3 subsection must contain one occurrence of each digit from 1–9

## Solution
```clojure
; Problem representation
; https://www.sudoku.ws/hard-20.htm
(def s1 '[1 - - - 7 - - 3 -
          8 3 - 6 - - - - -
          - - 2 9 - - 6 - 8
          6 - - - - 4 9 - 7
          - 9 - - - - - 5 -
          3 - 7 5 - - - - 4
          2 - 3 - - 9 1 - -
          - - - - - 2 - 4 3
          - 4 - - 8 - - - 9])

(defn rowify [board]
  (->> board
       (partition 9)
       (map vec)
       vec))
       
(defn colify [rows]
  (apply map vector rows))
  
(defn gridify [rows]
  (partition 9
             (for [row (range 0 9 3)
                   col (range 0 9 3)
                   x (range row (+ row 3))
                   y (range col (+ col 3))]
               (get-in rows [x y]))))  
               
(def initLogicBoard #(repeatedly 81 logic/lvar))

(defn init [[lv & lvs] [cell & cells]]
  "bind known number values to certain logic variable"
  (if lv
    (logic/fresh []
                 (if (= '- cell)
                   logic/succeed
                   (logic/== lv cell))
                 (init lvs cells))
    logic/succeed))

(defn solveSudoku [board]
  (let [lvars (initLogicBoard)
        rows (rowify lvars)
        cols (colify rows)
        grids (gridify rows)]
    (logic/run 1 [q]
               (init lvars board)
               (logic/everyg #(fd/in % (fd/interval 1 9)) lvars)
               (logic/everyg fd/distinct rows)
               (logic/everyg fd/distinct cols)
               (logic/everyg fd/distinct grids)
               (logic/== q lvars))))
               
(solveSudoku s1)              

(comment 
  "
  1 6 9 8 7 5 4 3 2
  8 3 4 6 2 1 7 9 5
  5 7 2 9 4 3 6 1 8
  6 2 5 1 3 4 9 8 7
  4 9 8 2 6 7 3 5 1
  3 1 7 5 9 8 2 6 4
  2 8 3 4 5 9 1 7 6
  9 5 6 7 1 2 8 4 3
  7 4 1 3 8 6 5 2 9 ")
  
```
