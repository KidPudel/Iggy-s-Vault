# Overview

- **Searching and Sorting**
    - Binary Search
    - Recursion
    - Bubble Sort
    - Merge Sort
    - Quick Sort
- **Maps and Hashing**
    - Maps
    - Hashing
    - Collisions
    - Hashing Conventions
- **Trees**
    - Trees
    - Tree Traversal
    - Binary Trees
    - Binary Search Trees
    - Heaps
    - Self-Balancing Trees
- **Graphs**
    - Graphs
    - Graph Properties
    - Graph Representation
    - Graph Traversal
    - Graph Paths
- **Case Studies in Algorithms**
    - Shortest Path Problem
    - Knapsack Problem
    - Traveling Salesman Problem
- **Technical Interview Tips**
    - Mock Interview Breakdown
    - Additional Tips
    - Practice with Pramp
    - Next Steps

# Searching

## Binary search

- **Explanation**
    
    _**Prerequisites**_: we have a sorted collection
    
    _**Purpose**_: find an element
    
    _**Method**_: compare middle element, if greater, then search second half of the list and so on
    

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/88a78218-7e40-4ffc-9039-175941aa3fbb/Untitled.png)

when array doubles its size, operation step is increases by 1

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/468f321e-b6ff-4b42-9885-032b53fb7d05/Untitled.png)

_**time complexity**_: 2^3 = 8 → log2(8) = 3 → 3 + 1 = 4 → log2(8) + 1 = 4 → **O(log2(n) + 1) → O(log(n))**

### **Advice: create a result tables to see patterns**

```kotlin
package algorithms

class Algorithms<T> {
    fun binarySearch(array: IntArray, value: Int): Int {
        // compare the middle, and check other half of array
        var start: Int = 0
        var end: Int = array.size - 1
        var middle: Int

        while (start <= end) {
            middle = (start + end) / 2

            if (value > array.elementAt(middle)) {
                start = middle + 1
            } else if (value < array.elementAt(middle)){
                end = middle - 1
            }

            if (value == array.elementAt(middle)) {
                return middle
            }

        }
        return -1
    }
}
```

## Recursion

Function that calls itself

> _People either love it or hate it, or hate it until they learn to love it a few years later_

Recursion is used when it makes the solution cleaner

```kotlin
fun searchForKey(box: Box): Key {
		for (item in box) {
				if (item.isBox()){ // Recursive case
						searchForKey(box)
				} else if (item.isKey()) { // Base case
						return item
				}
		}
		throw KeyNotFound()
}
```

### Base case and recursive case

to avoid infinite loop, every code with recursion must have:

1. recursive case (when a function calls itself)
2. base case when it doesn’t jump into itself

## Monotonic Stack

when we need to find the **_next_ greatest** element in the array we can use monotonic decreasing stack

it pushes elements that is lesser than previous, but when encounter greater element, pops element that is lesser. That way we store in decreasing order and at the same time we find only greatest elements that is next (_**because when we encounter it we pop those elements for which i-th element is greater**_) and we can find how many days we need to wait (difference)

The same think with finding next smallest elements, store in monotonic increasing order stack, add greater, and once encounter lesser, pop those that greater and find the difference in position for example.

```
fun dailyTemperatures(temperatures: IntArray): IntArray {
        val monotonicStack = Stack<Pair<Int, Int>>()
        val daysToWait = IntArray(temperatures.size)
        var lesser: Pair<Int, Int>
        for (i in temperatures.indices) {
            while (monotonicStack.isNotEmpty() && monotonicStack.last().second < temperatures[i]) {
                lesser = monotonicStack.pop()
                daysToWait[lesser.first] = i - lesser.first
            }
            monotonicStack.push(i to temperatures[i])
        }
        return daysToWait
    }
```

# Sorting

## Selection sort

You just go every time through the array and find greatest, and add to the top (or to the new one)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d80f9ef2-8ab5-43a9-9a6d-bc187f051298/Untitled.png)

## Bubble Sort

We iterate throw array and swap to elements (_without already sorted last elements_).

each iteration will _**bubble up**_ the largest element to end

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6940292e-88ea-4f94-b345-180972ae14c3/Untitled.png)

**iterations**: 5 elements, we did 4 iterations (n-1)

**comparison**: 5 elements, we did 4 (n-1)

result: (n-1)*(n-1) → n^2-2n+1 → n^2 → O(n^2)

Worst Case: O(n^2)

Average Case: O(n^2)

Best Case: O(n)

## Quicksort

Quicksort is using D&C.

Therefore we need to find base case and recursion case

### Base case

**base case** would be the simplest to sort and that is array that you don’t need to sort at all [], [5] with size 0 and 1

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7c1ab945-0f1f-4658-829a-ba57ed89fe0a/Untitled.png)

### Size of 2

if array is size of 2 then we just need to check:

if first elemen bigger then second → reverse

else → they are sorted

### Size of 3+

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/dd658084-00e4-4e05-a0cb-16bfd85d2d24/Untitled.png)

if array size is 3+, we need to choose a _**pivot**_ for instance 33 (**CHOOSE A RANDOM ELEMENT, to hit average case**)

then find elements that smaller and that is bigger than _pivot_

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5a34a4db-a5cc-4a20-9fce-f09061c7fce9/Untitled.png)

[10, 15], [33], []

now we need to apply this algorithm for two other arrays that are size of 2 and 0, which we already know how to deal with

quicksort([10, 15]) + [33] + quicksort([])

we can apply [inductive proof](https://www.notion.so/Proofs-b981268cc8644afa8ddb08e8fff30b60?pvs=21) here to proof that now we can sort with every array size

Sort 4

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a5f3aaff-59c9-4f00-81a5-2ab3522907ca/Untitled.png)

we have arrays size of 3 and 0 again

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bb516de4-e6e9-4107-a513-ad04b18646d4/Untitled.png)

Therefore we can sort 0, 1, 2, 3, 4, n..

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ad34929a-8aa2-4218-85d1-7e8e01d0c914/Untitled.png)

and we can choose any element as a pivot

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/992d7c30-a5f6-4b7c-b8b3-7ff0e27539fd/Untitled.png)

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

```kotlin
private fun dcSum(arr: List<Int>): Int {
    // base case -> simplest case -> list of 0
    return if (arr.isEmpty()) {
        0
    } else {
        // recursive case (find a way to get a base case)
        arr[0] + dcSum(arr.slice(1 until arr.size))
    }
}

private fun <T> dcSize(arr: List<T>): Int {
    return if (arr.isEmpty()) {
        0
    } else {
        1 + dcSize(arr.slice(1 until arr.size))
    }
}

private fun dcMax(arr: List<Int>): Int {
    return if (arr.size == 1) {
        // base case size is just one (greatest is this)
        arr[0]
    } else {
        // recursive case (compare current first with a next largest)
        val previousMax = dcMax(arr.slice(1 until arr.size))
        if (arr[0] > previousMax) {
            arr[0]
        } else {
            previousMax
        }
    }
}
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8910ae8d-fdce-48f1-8943-b37e67de5f54/Untitled.png)

# Top 5 Algorithms

## 1. Top k elements

_**shows up in:**_

1. _**find k**_ largest or smallest _**elements**_ numbers
2. _**find k**_ most frequent numbers of _**elements**_ in the array

Add first k elements _**to the heap**_, then add one more element (to have k + 1) → find smallest → remove, and add one to the heap again, find the smallest, repeat, at the end in the heap will be the largest numbers

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/18539a38-7ac8-4617-915a-07e1bd004ac3/Untitled.png)

## 2. Sliding Window

_**shows up in:**_

1. find largest substring with some rules (without repeating characters)
2. find a size of that substring

at the beginning of the array define _**two pointers (L, R)**_ the range of which is going to be a _window_, at each step add an element _**to hash set**_ and increment a pointer, if we’ve got a repeating element → update an answer, and move the left pointer until there is no repeating element, repeat (add, move and so on) until right pointer goes out of range of the array

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b1c34186-9a1a-41dd-b34d-b0140ce3d59e/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/45a39e1c-bf7b-46af-934b-f188f3237b28/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2832c5ab-446d-4950-9766-1652d2c114aa/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e72c1fe0-79d9-464d-8fb5-23c05bcb051b/Untitled.png)

## 3. Backtracking

Explore all possible solutions, by building them step by step

When we reach invalid state, we _**backtack**_ and start exploring other options

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0d1ba83f-3419-4ba6-82d7-7365342df13b/Untitled.png)

**Used in problems like:** Word Ladder, Permutations, Sudoku Solver

## 4. Dynamic Programming

In dynamic programming we are more thoughtful about the solutions we explore, by breaking the problems into smaller sub-problems

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6196fa58-f3fa-4546-82cc-32728e382620/Untitled.png)

We already have possible combinations, but we’ve given not all numbers

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/90359005-3b61-493b-a79e-493a6dca4ad0/Untitled.png)

We can go through the list and when the number is equals our last target, just add target

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a78a8b89-1ee9-42f9-a798-6ae2b0b8f383/Untitled.png)

after that we just look at the number - target → for example 6 (6 - 5 = 1 ⇒ 6 = [1, 5]), we look at one and see none, so do nothing

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5dbf8a2f-2ff0-4ae3-a457-c888e33a2471/Untitled.png)

now look at 7 ⇒ [2, 5]

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/05aab60f-5862-4c10-b0e9-6c88d27f230c/Untitled.png)

and we see that 2 contains some combination, so just add this combination + last target

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c752e57d-0643-4791-964c-7787a9adce3c/Untitled.png)

now do that for the rest

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7495116c-df3d-402d-ac11-ee8a9cfcfaf3/Untitled.png)

## 5.1. Depth For Search (DFS)

Start from a vertex and go on the branch as deep as possible until there is no unvisited vertices to explore, then backtrack and try to find another branch to explore

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e9e2b6ef-9a2f-4057-b063-893915b4aad8/Untitled.png)

**DFS implemented using Stack (last in first out)** because we need to explore neighbors of the last visited branch (vertex) first

_**(Closest is Topological Sort) Scheduling problems, cycle detection in graph, and solving puzzles with only one solution, such as maze or a sudoku game. Also involved in analyzing network, eg testing if a graph is bipartite (two parts)**_

## 5.2. Breadth For Search (BFS)

explore neighbors first (layers like in level order)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1ff1e647-3cec-4910-8b26-09bcd6f7df9f/Untitled.png)

**BFS implemented using Queue (first in first out)** because we need to explore neighbors of the first visited vertex first

_**Used to find a shortest path from vertex A to vertex B (Closest is Dijkstra’s Algorithm)**_