It is a [[singleton pattern]] object, that is allocated once on the [[heap]], accessed through the whole game, and [[static]] / lived as long as engine is running.
Low [[Memory fragmentation]]
[[cache locality]] is hight

> [!note] Autoload could be just a script/node or a whole scene. When you autoload a scene, all of its child nodes are automatically part of that scene's hierarchy, which keeps the structure self-contained and easier to manage.


Autoload should follow these **rules**:
1. Tracks all of its own data internally
2. Should exist in isolation
	1. not tightly coupled to scene-local state, not invading the data of other nodes, but rather do work on its own and let others call it, not the other way around.
	2. An **Autoload (singleton)** in Godot **should NOT depend on nodes in the scene tree**. 
	3. HINT: use [[signal]] for free data spreading as notifications and notification handlers

Autoload can be used if **satisfy following requirements *for the usecase***:
1. Needs to exist from start to end.
2. Must be globally accessed
3. Manage system with wide enough scope. Like weather system if your majority of the game is under the sky.
4. Are used by everything, regardless of context, and *not* belong to the individual component

> [!note] It all depends on what you want to do.
> For example: You want to build a scene manager. what is the goals? all or nothing solution, or rebuild from specific part? or mix of both?
> In conclusion: It is just depends on the scope of operation that you want to implement. If you are limited to only rebuild from parent, then go for child scene, otherwise you can operate with autoload. In my opinion it is a good separation of concerns.



---
Otherwise try to think of other solutions instead of the autoload, Or the following problems could arise

Should not use autoload:
1. For scene-specific or local data. Without it it is easier to maintain code
2. For resource management. Instead let scenes manage their own state, avoids 


# Not wide enough system, that we've decided to made an autoload
Problems:
1. Global access: Pollute code with a spread usage, which will cause bugs and will make them to hard to trace
2. Global resource allocations: If we have a pool of `AudioStreamPlayer` in audio manager, we can aim too low, and face bugs where we call it in several places at once, or aim too high and allocate more memory than we needed.
3. Global state: tight dependency on it, if it is not suited like not wide enough, it is simply not a reliable solution for it. If it will break, it will break in many places

Solutions:
Utilize the Godot engine architecture:
Each scene keeps as many `AudioStreamPlayer` as it needs within itself.
1. Now each scene access its own nodes, and place of bugs are easy to detect
2. We adjust as many memory as we need for each scene individually
3. Now since scene now manages its own state, problem will be local to this scene.
[[Call down, signal up]]

# We need a shared data or functionality, but the autoload is unnecessary
Instead of that we have [[static]] variables and functions, as well as [[Godot Resource]], that we can create, and then just include therefore share across.

Since autoload is not a part of the tree, you loose the ability of [[Call down, signal up]], since it is the outer entity.
And therefore also it pollutes your code, since you have only the ability to globally call logic of autoload across whole project.


[singleton autoload](https://docs.godotengine.org/en/latest/tutorials/scripting/singletons_autoload.html)
