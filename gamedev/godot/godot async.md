# Godot Threading & Async Quick Reference

## The Golden Rule

**Main thread only:** Scene tree, nodes, physics queries, rendering.  
**Threads safe:** Pure math, data processing, file I/O.

---

## 1. Threads (True Parallelism)

### Basic Usage

```gdscript
var thread: Thread

func start_work():
    thread = Thread.new()
    thread.start(my_function)

func my_function():
    # heavy work here
    call_deferred("on_done", result)

func on_done(result):
    thread.wait_to_finish()  # MUST call this to clean up
```

### Passing Arguments

```gdscript
thread.start(my_function.bind(arg1, arg2, arg3))

func my_function(arg1, arg2, arg3):
    # use args
```

### Returning to Main Thread

```gdscript
# From inside thread - these are safe:
call_deferred("method_name", arg1, arg2)      # calls on main thread next frame
emit_signal.call_deferred("my_signal", data)  # emit signal on main thread
```

### Thread Cleanup

```gdscript
# Always call when done, or in _exit_tree()
func _exit_tree():
    if thread and thread.is_started():
        thread.wait_to_finish()
```

---

## 2. Coroutines / Await (Cooperative, Single Thread)

**Not true async** - still blocks main thread, just spreads work across frames.

### Yield to Next Frame

```gdscript
func spread_work():
    for i in range(10000):
        do_thing(i)
        if i % 100 == 0:
            await get_tree().process_frame  # continue next _process
```

### Yield to Next Physics Frame

```gdscript
await get_tree().physics_frame  # continue next _physics_process
```

### Wait for Time

```gdscript
await get_tree().create_timer(1.5).timeout  # wait 1.5 seconds
```

### Wait for Signal

```gdscript
await some_node.some_signal
await $AnimationPlayer.animation_finished
var result = await $HTTPRequest.request_completed
```

### Wait for Another Coroutine

```gdscript
func parent():
    await child_coroutine()
    print("child finished")

func child_coroutine():
    await get_tree().create_timer(1.0).timeout
```

---

## 3. call_deferred

Queues a method call for the next idle frame. Safe to call from anywhere.

```gdscript
# Basic
call_deferred("method_name")

# With arguments
call_deferred("method_name", arg1, arg2)

# On another object
some_node.call_deferred("queue_free")

# Deferred property set
set_deferred("position", Vector3.ZERO)
```

**Common uses:**

- Freeing nodes during iteration: `node.call_deferred("queue_free")`
- Returning from thread to main thread
- Avoiding "can't modify during signal" errors

---

## 4. Mutex (Thread-Safe Shared Data)

```gdscript
var mutex: Mutex = Mutex.new()
var shared_data: Array = []

func thread_function():
    mutex.lock()
    shared_data.append(something)
    mutex.unlock()

func main_thread_read():
    mutex.lock()
    var copy = shared_data.duplicate()
    mutex.unlock()
    return copy
```

---

## 5. Semaphore (Thread Signaling)

```gdscript
var semaphore: Semaphore = Semaphore.new()
var thread: Thread

func _ready():
    thread = Thread.new()
    thread.start(worker)

func worker():
    while true:
        semaphore.wait()  # blocks until post() is called
        do_work()

func queue_work():
    semaphore.post()  # wakes up worker thread
```

---

## 6. Common Patterns

### Pattern A: Heavy Calculation → Main Thread Callback

```gdscript
var thread: Thread

func generate_terrain(size: int):
    thread = Thread.new()
    thread.start(_generate_terrain_threaded.bind(size))

func _generate_terrain_threaded(size: int):
    var heightmap = []
    for x in range(size):
        for z in range(size):
            heightmap.append(noise.get_noise_2d(x, z))
    call_deferred("_apply_terrain", heightmap)

func _apply_terrain(heightmap: Array):
    thread.wait_to_finish()
    # now safe to modify scene tree
```

### Pattern B: Chunked Processing (No Thread)

```gdscript
func spawn_many_items(items: Array):
    for i in range(items.size()):
        spawn_item(items[i])
        if i % 10 == 0:
            await get_tree().process_frame

func spawn_item(data):
    var instance = scene.instantiate()
    add_child(instance)
```

### Pattern C: Background Worker with Queue

```gdscript
var thread: Thread
var work_queue: Array = []
var queue_mutex: Mutex = Mutex.new()
var semaphore: Semaphore = Semaphore.new()
var running: bool = true

func _ready():
    thread = Thread.new()
    thread.start(_worker)

func _exit_tree():
    running = false
    semaphore.post()  # wake up thread so it can exit
    thread.wait_to_finish()

func _worker():
    while running:
        semaphore.wait()
        if not running:
            break
        
        queue_mutex.lock()
        var work = work_queue.pop_front() if work_queue.size() > 0 else null
        queue_mutex.unlock()
        
        if work:
            var result = process_work(work)
            call_deferred("_on_work_done", result)

func queue_work(data):
    queue_mutex.lock()
    work_queue.append(data)
    queue_mutex.unlock()
    semaphore.post()

func _on_work_done(result):
    # handle result on main thread
    pass
```

---

## 7. What You CAN'T Do From Threads

- Access/modify nodes
- `get_tree()`
- `get_world_2d()` / `get_world_3d()`
- Physics queries (raycast, shapecast, etc.)
- `add_child()` / `remove_child()`
- Modify resources that are in use
- Most rendering operations

---

## 8. When to Use What

|Situation|Solution|
|---|---|
|CPU-heavy math, no scene access|Thread|
|Need to touch scene during work|Chunked await|
|Physics queries in bulk|Chunked await|
|File I/O|Thread|
|Network requests|HTTPRequest node (built-in async)|
|Wait for animation/timer|await signal|
|Delete node during iteration|call_deferred("queue_free")|
|Light work, simple code|Chunked await|
|True parallelism needed|Thread|

---

## 9. Quick Gotchas

1. **Always `wait_to_finish()`** - memory leak otherwise
2. **Threads can't touch nodes** - use call_deferred
3. **await doesn't parallelize** - just yields, still one thread
4. **Mutex everything shared** - or get race conditions
5. **Check `is_started()`** before `wait_to_finish()` in _exit_tree
6. **Don't await in threads** - await is main thread only