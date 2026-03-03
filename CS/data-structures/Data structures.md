# Overview

- **List-Based Collections**
    - Lists/Arrays
    - Linked Lists
    - Stacks
    - Queues
- Maps and Hashing

> Collection is a group of things
# Lists/Arrays

**List** - it’s a sequential immutable memory region (random access, because under the hood is ArrayList)

```kotlin
val list = listOf(1, 2, 3, 4, 5)
println(list[0])
```

**Array** - it's a sequential fixed-size memory region storing the items Arrays are optimized for primitives: there are separate `IntArray`, `DoubleArray`, `CharArray` etc. which are mapped to Java primitive arrays (`int[]`, `double[]`, `char[]`),

```kotlin
val array = arrayOf(1, 2, 3, 4, 5)
// values can be changed
array[0] = 6
// but size is fixed
println(array.size)
// output
// 5
```

**ArrayList** - it’s a list where it is fast to **get values** O(1) 😃, but it’s tedious to insert/delete at _**not end**_ position _(shifts values)_ worst case is O(n) dynamically allocated (from the main memory, heap)😔

```kotlin
val arrayList = arrayListOf(1, 2, 3, 4, 5)
// values can be changed
arrayList[0] = 6
// add values
arrayList.add(index = arrayList.size, element = 11)
// size is not fixed
println(arrayList.size)
println(arrayList.joinToString(separator = ", "))
// output
//6
//6, 2, 3, 4, 5, 11
```

[Linked List](https://www.notion.so/Data-structures-52d16e384c42440086a9acfd28a04cbd?pvs=21) - is basically an opposite to array list

> If you don't care about concrete implementation use `List` (probably 99% of the cases 🙂). If you do care about implementation use `ArrayList` or `LinkedList` or any other concrete implementation.

_However, Python’s naming convention doesn’t provide the same level of clarity that you’ll find in other languages. **In [Java](https://realpython.com/oop-in-python-vs-java/)/Kotlin, a list isn’t just a `list`—it’s either a `LinkedList` or an `ArrayList`**. Not so in Python. Even experienced **Python developers sometimes wonder whether the built-in `list`type is implemented as a [linked list](https://realpython.com/linked-lists-python/) or a dynamic array**._





# Hash tables (Maps/Dictionary)

Hash tables are great when you want to:

- create a mapping from one thing to another (relationship)
- lookup something (_**ger really fast!!!**_)
- finding duplicated
- cashing something

### Usage

Suppose we at grocery store, we can search in a book an item

- if not sorted then O(n)
- if sorted then we can use binary search O(logn)

But if we had hash table (Maggie) that could help us get a price of an _apple_ for example, that would be _**O(1)**_

![[Pasted image 20240617154604.png|500]]

### To create

To create a hash table we need to use hash function (mapping key to value) `“apple” to 0.67`
![[Pasted image 20240617154638.png|500]]

Requirements

- Consistent (if apple then return 0.67 every time)
- Best is to different key there is different value (not always return “1”)

So you “feed” hash function with a new key-values and hash function outputs an index where to put in array

![[Pasted image 20240617154712.png|500]]
After you’ve filled the entire array, you give hash function “avocado” and it tells that it’s at the index of 4!


- Hash function returns the same index for a key each time (first time you feed it with a string → it adds to the array and returns an index, next time you feed with the same string it will just return an index)
- Hash function gives each key different index and put a value in that index
- Hash function knows about array size (no index out of bounds)

---

> _**Hash function + Array == Hash table**_

---

**Store key-values pairs**. In Python, dictionaries, while in Kotlin, maps

```kotlin
val map = mutableMapOf<Int, String>()
    map[0] = "first"
    map[1] = "second"

    println(map.values.firstOrNull {it == "second"} ?: "none")
```

### Preventing duplicate entries

One person can vote once, if there is a lot of people in the list, it can be hard

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9be4965b-93bf-4bea-bcf7-8c3cf2dc9432/Untitled.png)

so use hash, if name is returns something, then they voted, otherwise let them vote

### Cashing things for faster access and less work

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ebe5d0ad-d2d2-41bb-8acc-b792b58e1471/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a0a2824c-0313-4407-81f2-502fe72a7cee/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5e9b055d-12bc-4454-9273-9c38eea6c382/Untitled.png)

### Collisions

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/08eeeb35-3c5f-44d9-8d3d-8b0cf2210869/Untitled.png)

two keys have been assigned the same slot

**Simplest solution:**

start a linked list

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/edb990fa-c96e-44e4-9542-2204102d9ffe/Untitled.png)

finding bananas is still fast, finding apples is a little slower, but what if this linked list is getting bigger?

There could be only a’s groceries, meaning:

1. there will be a giant linked list
2. all other memory slots will be wasted

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/217c88f6-21ed-4119-b7e2-4ed70df55e68/Untitled.png)

two lessons:

1. **hash function is very important**, ideally hash function map keys evenly in the hash
2. long linked list = slow hash table ⇒ hash function should be good to avoid this

### Performance

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d157c636-377c-431e-b156-5985a90c0c7f/Untitled.png)

To not hit a worst-case performance → avoid collisions

to avoid collisions, you need:

1. A low load factor
2. A good hash function

---

**Load factor:**

Load factor is → number of elements ➗ number of slots

Load factor measures how many slots are available

hash table use array for storage, so you can count the numbers of elements

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1f4f3197-4da1-4c3e-b4de-4277bb1956e7/Untitled.png)

having a load factor more than one → there is more elements than slots

Once load factor starts to grow, we need to:

1. add more slots → _**resize it | Rule of thumb - create a new array that is twice the size once load factor is greater than 0.7**_
2. Then, re-insert the elements using the hash function

> So with a lower load factor, you’ll have lower chance of collision

Resizing expensive, but averaged out, hash table takes O(1) when resizing

---

**Good hash function:**

Good hash function - distributes elements evenly

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ffe1850b-cac2-4099-8213-81507623f01e/Untitled.png)

Bad hash function - groups elements together → creates a linked list

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c2d8988d-4080-48f4-a89b-038885311eee/Untitled.png)

### Example

```kotlin
val votedList = mutableMapOf<String, Boolean>()
// ...

if (votedList.get("iggy") ?: false) {
		println("already in the list")
} else {
		votedList.put("iggy", true)
}
```

# Sets

Data structure that **store unique elements**. (Python build in support for sets, Kotlin’s Sets is a part of the collections framework

**usecase: store unique, as for hashset quick lookup O(1)**

### hashsets (unordered)

hashsets are also using hash function to make it work

but instead of key: value there is just key, so that way we also getting unique values

### sets (ordered)

with balanced tree 95% of the time black-red tree

There is a setOf creates a LinkedSetOf that is a linked list _(if order of operation matters)_

And mutableSetOf or HashSet that is using hash function

```kotlin
val setOfFlowers: MutableSet<String> = mutableSetOf("Sunflower", "Sunflower", "Lavender", "Rose")
// or
val setOfFlowers = hashSetOf("Sunflower", "Sunflower", "Lavender", "Rose")

println(setOfFlowers.joinToString(separator = ", ", prefix = "Flowers: "))
setOfFlowers.add("Lavender")
println(setOfFlowers.joinToString(separator = ", ", prefix = "Flowers are unique: "))

Flowers: Sunflower, Lavender, Rose
Flowers are unique: Sunflower, Lavender, Rose
```


# Stacks

Stack - Data structure that implement the **Last-in-First-out (LIFO)**

- top element called - head
- bottom element - tail

**usecase: get the current last (stored previously) that we need later (_now get the resent, then deal with more outer things, like brackets {([now the current last most nested is ] then ) then }_ )**

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8db94b6b-14ec-4500-b142-15f7c65d75b1/Untitled.png)

to add top element - push O(1)

to delete - pop O(1)

_Computer also uses a stack - “call stack”, when we call a function it allocates box of memory for function call with all parameters, inside a function it calls another function and new box gets pushed to the top of a stuck, and when program returns to the previous function, the second one gets popped_

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5aa6fba2-8b11-486f-939e-ae3034b1ee29/Untitled.png)

> The stack that is saving a variables for a multiple functions is called - call stack

```kotlin
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

# Queues

Queues- **First-in-First-out (FIFO)**

- oldest element called - head
- newest called - tail

to add element to the tail - enqueue

to delete head element - dequeue

to lookup first element - peek

## Deque (double ended queue)

you can threat it like stack and queue

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a2096b54-8bf3-4186-a97b-d5dff52286b5/Untitled.png)

```kotlin
val deque: ArrayDeque<Int> = ArrayDeque()
deque.addLast(1)
deque.addLast(2)
deque.addLast(3)
println(deque.joinToString(separator = ", "))
println("what, why is he first ??!")
deque.addFirst(4)
println(deque.joinToString(separator = ", "))
println("come on..")
deque.removeFirst()
println(deque.joinToString(separator = ", "))
println("i'm tired waiting, i'm leaving")
deque.removeLast()
println(deque.joinToString(separator = ", "))

1, 2, 3
what, why is he first ??!
4, 1, 2, 3
come on..
1, 2, 3
i'm tired waiting, i'm leaving
1, 2
```

## Priority Queue

Delete the element with the highest priority, if priority is equals, then delete the oldest one (head)

# Linked Lists
[[linked list]]


# HashMap vs Map & HashSet vs Set

- hash- using hash functions and they are unordered because key is given an address by which it access a value → it’s inserted in a random place in memory
- others are ordered they are using balanced tree 95% is black-red tree

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/c312b26c-2049-4a41-8dca-aa90f900b9e4/0fba1a3e-b800-4188-af45-335a23ddb97e/Untitled.png)
# Top 5 asked data structures

## Heap

_**usecase: get sorted right away!!!**_

Tree based DS to store partially sorted set of elements

_It’s implemented using the array_

### When to use?

When we need to find largest or smallest element (heap makes it very easy, cus it’s at the top)

We can use heap to implement priority queue

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/437f3505-55fa-4465-922c-7af2a127377e/Untitled.png)

_**Algorithms where appears on**_: Heap sort. Dijkstra’s Algorithm, Median of Stream

### Types of heap

- Max Heap (root or top most element is the greatest of all elements)
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ae1f2056-1375-4bc4-a125-8161e7211c25/Untitled.png)
    
- Min Heap
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ce0c2b85-8f81-44a2-87e9-60fd785bfe90/Untitled.png)
    

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/58f18433-6c2f-4f95-a5dc-fb3005732034/Untitled.png)

_**Children of element n could be found at (2n + 1 and 2n + 2)**_

- convert any array to heap with algorithm heapify
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c6347716-3490-4aee-87c2-cbfc540c9565/Untitled.png)
    

## Binary Tree

Each element stores a value and the link to the left and the right child

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8512d7dc-3d6f-477c-8881-4ae356a70bbb/Untitled.png)

### When to use?

_**Use case**_: Database indexing, Sorting algorithms. Decision trees

_**Algorithms where appears on**_: when asked to think about binary tree

_**They are perfect candidates for recursion**_

Types of traversal (process of visiting all the nodes of a tree)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8c623972-ff29-42c3-88e5-7afd3b7bddfc/Untitled.png)

## Hash Tables/HashMap

Provides fast access to a data based on a key,

_but quck lookups use extra space_

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6fdcdac3-4761-4b04-9a23-5a96980d614a/Untitled.png)

### When to use?

For example, we need to find an index of an element which value + current value is equals ⇒ 100

100 - current value, do this until result is not equals to the value in the hashtable (100 - 92 == 8) → 92 (3 index) , 8 (1) ⇒ (1. 3)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/07cc0f45-e106-467a-87f0-bd81b58f32d7/Untitled.png)

## Stack & Queue

Stack: Last added to the stack removed first

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fae60e10-5927-4e2b-9ee5-e6a5b51122c4/Untitled.png)

Queue: First added to the queue is removed

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9bdd81c9-6fe7-4eaf-b985-ea5f955678ff/Untitled.png)

They are both implemented with an array or linked list

### When to use Stack?

When you have previously added elements, and we need to get latest, use Stack

## Graphs

Data structure that consists of vertices and edges that connect those vertices

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0d67ad13-df3f-42a2-a8e7-8b1c583ba6a9/Untitled.png)

There is directional and undirectional graphs

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c897937c-886c-4fc3-9ade-2973efd5740f/Untitled.png)

### When to use?

When you spot a relationship between entities

Bus roads that goes through a bus stops and we need to find a route between stop 15 and 12 with a minimum bus changes

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0e27fbdc-c7d0-4f61-b230-fbdf5d4eff3f/Untitled.png)

### Graph concepts

Graph Search

1. Depth for search (for exploration)
2. Breadth for search (find a shortest path between two nodes on the graph)
3. Dijkstra’s Algorithm (find a shortest path between two nodes on the graph)