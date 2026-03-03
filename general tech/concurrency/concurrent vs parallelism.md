# Concurrency as customer


![[Pasted image 20240312110822.png]]
![[Pasted image 20240312111944.png]]
![[Pasted image 20240312112003.png]]
They could do other stuff like talk
# Parallelism as the customer
![[Pasted image 20240312112154.png]]
![[Pasted image 20240312112209.png]]
Stack waiting
![[Pasted image 20240312112223.png]]


# where parallelism shines (as the cleaner)
Cleaning a dirty house with a lot of rooms.

You need to clean a house, here is no waiting, you just need to clean it

Here *concurrency, which allows for executing for you multiple things by switching and waiting*, *will take the same amount with parallelism that cleans one by one, since there is no other tasks to do and no waiting to do something other*.
But where the parallelism shines, is that with this approach, **we could bring for example 8 cleaners (threads), and clean them together !!! (8 times faster)**
