# Coroutines: Concurrent Logic

A coroutine is a thread that allows a function to run concurrently with other code without yielding the entire script. If you need to run a long-running task (like a countdown timer, a moving platform, or a patrol path for an NPC) without stopping the rest of your script, you use a coroutine.

Roblox's task scheduler handles running coroutines, so they are a safe and efficient way to manage multiple logical flows at once.

*   **`coroutine.create(func)`**: Creates a new coroutine with the given function but does not start it.
*   **`coroutine.resume(co)`**: Starts or continues the execution of a coroutine.
*   **`coroutine.wrap(func)`**: A convenient alternative. It returns a function that, when called, will start the coroutine.

**Example: A Part that Fades and a Message that Prints Simultaneously**

```lua
local part = workspace.MyPart

local function fadePart()
    for i = 1, 10 do
        part.Transparency = i / 10
        task.wait(0.1) -- 'task.wait()' is the modern, more precise version of 'wait()'
    end
    print("Fading complete.")
end

local function printMessages()
    for i = 1, 5 do
        print("Message number", i)
        task.wait(0.25)
    end
end

-- Wrap the functions to turn them into coroutine starters
local startFading = coroutine.wrap(fadePart)
local startPrinting = coroutine.wrap(printMessages)

-- Start both tasks. They will now run concurrently, not one after the other.
startFading()
startPrinting()

print("Main script thread finished.")
```
In the example above, you will see the messages printing in the output while the part is simultaneously fading. Without coroutines, the script would have to wait for the entire fade sequence to finish before it could start printing the messages.