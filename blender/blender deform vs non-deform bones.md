Control bones don’t deform the mesh — **only DEF bones do**.
DEF stands for deform


| **Feature**                 | **Deform Bone**                 | **Non-Deform Bone**                                                                           |
| --------------------------- | ------------------------------- | --------------------------------------------------------------------------------------------- |
| Affects mesh shape?         | ✅ Yes (deforms vertices)        | ❌ No                                                                                          |
| Used in weight painting?    | ✅ Yes                           | ❌ No                                                                                          |
| Appears in export?          | ✅ Often kept (if skinned)       | ✅ Sometimes (but does nothing)                                                                |
| Used for animation control? | ❌ Not directly (usually driven) | ✅ Often used for control (IK, FK)                                                             |
| Usable in Godot runtime?    | ✅ Yes (if animated or moved)    | ⚠️ Only if you manually drive them and use them to attach things — they don’t deform the mesh |

Rigify is a **control rig system** with:
- Control bones for animators
- Drivers and constraints
- Many helper bones that **don’t deform the mesh**

### **When you export:**
- Godot sees all the bones, but it **doesn’t know about constraints**.
- So moving an IK controller (like upper_arm_tweak.L) does **nothing**, because:
    - The constraints are lost
    - It’s non-deform
    - The mesh doesn’t react

  

So people try to move these bones in Godot and are confused why **nothing happens** — that’s why GameRigTools exists.


## **✅ Summary of Advice**

- ✔️ **Deform bones = good**. These are the bones you should animate or move in Godot.
- ❌ **Non-deform bones = for Blender only**. They’re useful for animation **in Blender**, but not for runtime control.
- 🧰 Use **GameRigTools** if:
    - You want to use Rigify in Blender.
    - But you want a **clean, minimal export** to Godot with only deform bones.
- 🦴 Or just **make your own minimal rig** if you don’t need full Rigify IK/FK switching.












## 1. The Meta-Rig (Rigify’s control input)

This is what you create first when using Rigify:
- You go to **Add → Armature → Human (Meta-Rig)**.
- It’s a human-like stick figure with smart bone names.
- This is **not deforming the mesh**.
- You **do not animate** this rig.
- You use it as a **blueprint** to generate a real rig.

> Think of it as a **template rig**. Just an interface to auto-generate something more complex.

---


## 2. The Generated Rigify Rig (Control Rig)

When you click **“Generate Rig”** in Rigify, Blender does:
- Builds a **complex animation rig** with:
    - **Control bones** (like CTRL-hand.L, IK-leg.R)
    - **Helper bones** (like MCH-forearm.L, ORG-shoulder.L)
    - **Deform bones** (like DEF-upper_arm.L, DEF-thigh.R)
- Adds **constraints and drivers**:
    - IK chains
    - Copy transforms
    - Rotation limits

You use this **complex rig** to **animate your character in Blender**.

But:
⚠️ **Only the DEF- bones are deforming the mesh!**

Everything else is just for controlling animation behavior **inside Blender only**.

---

## 3. The Deform Rig (GameRigTools / Export Rig)

This is what **GameRigTools** (or manual work) gives you when preparing for game engines:
- It **strips away** all unnecessary bones:
    - Removes CTRL-, MCH-, ORG-, etc.
    - Keeps only the **DEF- bones** that actually deform the mesh.
    - Optionally **bakes animation** from the control rig to the DEF- bones.
- Renames or flattens bone hierarchy for compatibility.
- Ensures it is **game engine friendly**:
    - No constraints
    - No drivers
    - Only bones that affect vertex weights
  

In short: this rig is **clean**, **simple**, and **engine-compatible**.