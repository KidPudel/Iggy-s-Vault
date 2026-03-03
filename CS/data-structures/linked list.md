In Python, linked lists can be implemented using objects and references, while in Kotlin, they are part of the collections framework.

**Linked List** - Data structure that consists of sequence of nodes, where each node consists of an **element and reference to the next node**. It’s easy to insert O(1) 😃, just add a link to the next element and the element itself. But to access element, you have traverse through it O(n) 😔

First node is head

Last node is tail

Also there is a doubly linked list (store: element, pointer to the next, and previous element)

Kotlin implements Doubly

```kotlin
val linkedList = LinkedList<Int>(listOf(1, 2, 3, 4, 5))
// values can change value
linkedList[linkedList.size - 1] = 8
// add values, size is not fixed
linkedList.add(6)
println(linkedList.joinToString(separator = ", "))
// 1, 2, 3, 4, 8, 6
```
