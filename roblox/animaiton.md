The core of Roblox animation is three objects and one rule about who controls playback:

**Objects:**
1. **Animation** – just a reference to an animation asset (AnimationId).
2. **Animator** – the component that actually _plays_ animations.
3. **AnimationTrack** – a handle returned when you load an Animation into an Animator; this is what you play, stop, adjust speed on.

**Rule:**
Animations only play on a **Humanoid** (through its Animator) or on an **AnimationController** (for non-character rigs).

---
### **How it works in practice**

**Humanoid characters** already contain an Animator.
So you do:
```lua
local humanoid = character:WaitForChild("Humanoid")
local animator = humanoid:WaitForChild("Animator")

local animation = Instance.new("Animation")
animation.AnimationId = "rbxassetid://YOUR_ANIMATION_ID"

local track = animator:LoadAnimation(animation)
track:Play()
```

That’s it. track is your animation instance.

---

### **AnimationTrack is where all control lives**

On the track you can:
```lua
track:Play(fadeTime, weight, playbackSpeed)
track:Stop(fadeTime)
track.Looped = true
track.Priority = Enum.AnimationPriority.Action
track:AdjustSpeed(number)
```

**Important:** Animation priority determines which animation wins when multiple play.
Common priorities:
- Idle
- Movement
- Action
- Core (lowest)

A run animation should have **Movement** priority.
An attack animation should have **Action** priority.

---

### **Replication rules (critical)**

- **Server plays animations → all clients see it.**
    (Server owns the “truth” of character animations.)
- **Client plays animations on its own character → replicates to others.**
    (Because your client owns your character’s humanoid.)
- **Client cannot animate another player’s character.**
    (Those calls will not replicate.)

---

### **Non-character rigs (e.g., animated NPC, animated model)**

If there is no Humanoid, you must add an **AnimationController** and an **Animator** manually:
```lua
local controller = Instance.new("AnimationController")
controller.Parent = model

local animator = Instance.new("Animator")
animator.Parent = controller
```
Then load/play animations the same way via animator:LoadAnimation.

---
### **The summary you actually use**

**To play animation:**
- Get Animator.
- Create Animation.
- LoadAnimation.
- Keep AnimationTrack.
- Control via Play, Stop, Priority, AdjustSpeed.

**To ensure replication:**
- Server or the owning client should play the animation.
