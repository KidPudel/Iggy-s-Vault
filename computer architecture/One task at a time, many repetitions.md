
You need to chop 1000 onions. [[Big O notation]], think about the speed of the work
For that we also need to remember where the tools are stored for us / [[CPU]] to utilize it.
Everything has to be stored somewhere, moving tools and ingredients is expensive, and interputs from the main goal of doing the task.

![[Pasted image 20250512191336.png|1000]]

Think about your kitchen efficiently. [[i-cache]], [[d-cache]]

NOTE: i cache and d cache are separate which allows for parallel access of both

Utilize them fully until we don't need it, to then let it be replaced with the next tool/instruction and ingredient/data, to not constantly encounter [[cache miss]] and override [[CPU cache]].

# Summary
Do one task multiple in a bulk, while you still have:
- Tools in [[i-cache]]
- Ingredients in [[d-cache]]


# Techniques
- [[Batch processing]]
- [[Loop flattening]]
- [[Out of band]]