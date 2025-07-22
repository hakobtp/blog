# Relationship between exponential and logarithms

The 3 points plotted below are on the graph of $$y=b^x$$ for $$b=4$$.

$$

  \begin{array}{c|c}
    x & y = b^x \\
    \hline
    0 & 1       \\
    1 & 4       \\
    2 & 16      \\
  \end{array}

$$




**The Python code**

```python
import matplotlib . pyplot as plt
import sympy as sp

def createPlot ( fromTick , toTick ) :
    # Create an empty plot
    plt.figure ()

# Set the x and y- axis limits
9 plt . xlim ( fromTick , toTick )
10 plt . ylim ( fromTick , toTick )
11
12 # Define ticks for both axes
13 ticks = list ( range ( fromTick , toTick + 1) ) # Include toTick in ticks
14
15 # Set the x and y- axis ticks
16 plt . xticks ( ticks )
17 plt . yticks ( ticks )
```