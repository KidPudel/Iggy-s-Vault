Hot data is frequently accessed, meaning we need it in [[CPU cache]] to have [[cache hit]]
Cold data is rarely accessed, meaning we separate it from hot data, so we don't waste [[CPU cycle]]s on repeated access of [[cache line]] because of its limit.
This way we decreasing the amount of cycles needed to grab the whole data we need.
Grab only needed. not less not more, the ideal is to fully, efficiently utilize [[cache line]]


> **TL;DR: Don't waste the cache time**

For example: if you access a data structure frequently, best is to organize it the way it would certainly fit 64 byte, and if you have much less than it, then get multiple things

![[Pasted image 20250512185320.png|1000]]
![[Pasted image 20250512185907.png|1000]]
