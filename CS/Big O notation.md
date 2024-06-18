big O notation measures time complexity AND memory complexity

### Common Big O notations

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0633d974-df54-4d0e-b8d4-e3e236bf4f88/Untitled.png)

But actually O(n) is c*n where c is fixed amount of time

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/333c2efc-05e4-41ff-91e1-cf6b0f6d81d5/Untitled.png)

e.g. if we suspend each operation for 1 second then this constant will be much bigger, but it almost never matters

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/67667b60-bad3-4f99-9a15-cbf5730b0724/Untitled.png)

But sometimes it matters, like in comparison between quicksort and merge sort

merge sort is O(nlogn)

qucksort is O(n logn) average-best where the worst case is O(n2), but it’s very rarely

### Worst case

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/25338c70-957b-4677-be3a-12d2ed02c49b/Untitled.png)

so each level takes O(n) and height of call stack is O(n) means O(n^2)

### Best (average) case

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5633e8a7-3b92-4a70-903b-4775c1cbecef/Untitled.png)

each level takes O(n) and height of call stack is O(logn) ⇒ O(n logn)

### Time efficiency

if we have a program that runs once

```kotlin
fun main() {
    val secretCode: String = "the meaning of the life is .."
}

or 

fun main() {
    val secretCode: String = "the meaning of the life is .."
    println("$secretCode 42?")
}
```

this code has O(1) and O(2) - it is constant

```kotlin
fun main() {
    val secretCode: String = "the meaning of the life is .."
    repeat(42) {
        println("waiting")
        println(42 - it)
    }
    println("$secretCode 42?")
}
```

this code has O(2n + 2)

2^3 = 8 → log2(8) = 3 → O(log2(n))

### Approximation

We have:

1. **Worst case scenario**
2. **Average**
3. **Best**

But we can just simplify it to

O(2n + 2) → O(n) - some number of computation must be performed for each line (in loop)

### Memory complexity

if we would allocate 3 arrays

then memory complexity is O(3)

### Example 1

**We iterate over every `manatee` in the `manatees` list with the for loop**. Since we're given that `manatees` has n elements, our code will take approximately **O(n)** time to run.

### Example 2

We look at two specific properties of a specific `manatee`. **We aren't iterating over anything** - just doing constant-time lookups on lists and dictionaries. Thus the code will complete in constant, or **O(1)**, time.

### Example 3

**There are two for loop**s, and **nested for loops** are a good sign that you need to **multiply two runtimes**. Here, for every `manatee`, we check every property. If we had 4 `manatees`, each with 5 properties, then we would need 5+5+5+5 steps. This logic simplifies to the number of `manatees` times the number of properties, or **O(nm)**.

### Example 4

Again we have nested for loops. This time we're iterating over the `manatees` **list twice - every time we see a `manatee`, we compare it to every other `manatee`'s age.** We end up with **O(nn)**, or **O(n^2)** (which is read as _"n squared"_).

Throughout the course, you can reference the **[Big-O Cheat Sheet](http://bigocheatsheet.com/)** to keep track of time complexities for many of the algorithms and data structures we study.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2eff061d-f562-40d2-86d5-3e3aedd7f3e0/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cafff427-15f8-4b29-a1f0-05248be1823c/Untitled.png)
