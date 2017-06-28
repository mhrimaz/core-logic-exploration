# core.logic Exploration
Exploration in Clojure core.logic

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

