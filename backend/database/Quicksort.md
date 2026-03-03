General-purpose sorting algorithm

Quicksort is [[Divide and conqure]] algorithm.
There are 2 approaches Hoare (Left and Right) and Lomuto (Swap marker)
In Hoare approach It works by choosing a pivot, and dividing between this pivot into 2 sub-arrays, and do the same recursively
we take two pointers and looking for
- greater or equals to the pivot from the left
- smaller that pivot from the right
meaning the wrong ordered, and swap them until greatest left gone out of bounds (has greater index than smallest right) and repeat (in reverse (from smallest to greater arrays since it is a recursion))

Therefore we need to find base case and recursion case

### Base case

**base case** would be the simplest to sort and that is array that you don’t need to sort at all [], [5] with size 0 and 1

### Size of 2

if array is size of 2 then we just need to check:

if first elemen bigger then second → reverse

else → they are sorted

### Size of 3+


if array size is 3+, we need to choose a _**pivot**_ for instance 33 (**CHOOSE A RANDOM ELEMENT, to hit average case**)

then find elements that smaller and that is bigger than _pivot_

[10, 15], [33], []

now we need to apply this algorithm for two other arrays that are size of 2 and 0, which we already know how to deal with

quicksort([10, 15]) + [33] + quicksort([])

we can apply [inductive proof](https://www.notion.so/Proofs-b981268cc8644afa8ddb08e8fff30b60?pvs=21) here to proof that now we can sort with every array size

Sort 4

we have arrays size of 3 and 0 again


Therefore we can sort 0, 1, 2, 3, 4, n..


and we can choose any element as a pivot


```kotlin
private fun myQuicksort(arr: List<Int>): List<Int> {
    // base case (we don't need to sort) 0 or 1
    if (arr.size < 2) {
        return arr
    } else {
        // recursive case (find a way to get to base case) -> choose a pivot, sort greater and smaller array with quicksort (recursion moment)
        val pivot = arr[0]
        val greater = mutableListOf<Int>()
        val smaller = mutableListOf<Int>()
        // get greater and smaller
        for (number in arr) {
            if (number > pivot) {
                greater.add(number)
            } else if (number < pivot) {
                smaller.add(number)
            }
        }
        val sortedGreatest = myQuicksort(greater)
        val sortedSmallest = myQuicksort(smaller)

        return sortedSmallest + pivot + sortedGreatest
    }

}
```

# Recursing algorithms

When a problem can’t be solved by any algorithm, good algorithmist don’t give up - they have a toolbox fool of techniques to come up with their own solution. Below is the list of these techniques

## Divide-and-conquer strategy

When you encounter a new problem you ask yourself: “can i solve this problem useing D&C?”

One D&C algorithm is Quicksort

D&C algorithms are recursive, so to solve a problem using D&C, here are the following steps:

1. Figure out the base case. Should be the simplest as possible
2. Divide or Decrease your problem until it becomes a base case.

You have a farm, we need to find a largest square that works for entire field

1. Find the simplest base case (divide)
    
2. Find a way to get to that base case (conquer)
    
3. the simplest is when smaller side == bigger side / 2 ⇒ _**that is the base case**_
    
4. to get to that we try to find the biggest square. | By Euclid’s Algorithm → the biggest box that works for the space, is the biggest box that works for entire field
    

- **Steps**
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/46124215-4760-48f4-b494-bef6fd45713e/Untitled.png)
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e739dd4d-763b-4b5f-bfc8-07cdd861d719/Untitled.png)
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/94fde08c-7af6-485b-8066-1aef3f8b1c02/Untitled.png)
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/57df04ff-fc4f-4631-a3a7-fb898982dc00/Untitled.png)
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/15db1cdf-8c2a-4190-9bc6-918d663e3331/Untitled.png)
    

> _**D&C isn’t a simple algorithm, it’s a way to think about a problem**_

find a sum of list using D%C