When you encounter a new problem you ask yourself: “can i solve this problem using D&C?”

One D&C algorithm is [[Quicksort]]

D&C algorithms are recursive, so to solve a problem using D&C, here are the following steps:

1. Figure out the base case. Should be the simplest as possible, case to terminate the recursion
2. Divide or Decrease your problem until it becomes a base case.

You have a farm, we need to find a largest square that works for entire field

1. Find the simplest base case (divide)
    
2. Find a way to get to that base case (conquer)
    
3. the simplest is when smaller side == bigger side / 2 ⇒ _**that is the base case**_
    
4. to get to that we try to find the biggest square. | By Euclid’s Algorithm → the biggest box that works for the space, is the biggest box that works for entire field
