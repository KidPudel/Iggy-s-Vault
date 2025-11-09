# CFrame: Position and Orientation

In Godot, you have the `Transform3D` type to represent an object's position and rotation. The Roblox equivalent is the **`CFrame`** (Coordinate Frame). It is one of the most important data types for any 3D manipulation.

A `CFrame` is more than just a `Vector3` position. It is a data structure that contains both:
1.  **Position**: A `Vector3` value representing its location in the 3D world.
2.  **Orientation**: A 3x3 rotation matrix that describes its orientation (rotation) on the X, Y, and Z axes.

You will use `CFrame` to move and rotate parts in a way that `Position` and `Orientation` properties cannot. Modifying a part's `CFrame` is the standard way to teleport objects, create procedural movement, and aim turrets.

#### Key Operations and Concepts

*   **Creating a CFrame**:
    ```lua
    -- Creates a CFrame at the world origin (0, 0, 0) with no rotation
    local newCFrame = CFrame.new(0, 10, 5) -- At position (0, 10, 5)

    -- Teleport a part by setting its CFrame
    part.CFrame = newCFrame
    ```

*   **Object Space vs. World Space**: This is a critical distinction.
    *   **World Space**: An absolute position or direction in the game world. "Move 5 studs up on the world's Y-axis."
    *   **Object Space**: A position or direction *relative* to the part's own orientation. "Move 5 studs forward, in whatever direction the part is currently facing."

*   **Transforming in Object Space**: You use multiplication (`*`) to combine CFrames. The order matters. `CFrame1 * CFrame2` means "take CFrame1 and then apply the transformation of CFrame2 relative to it."
    ```lua
    -- Move a part 5 studs UP relative to its own orientation
    part.CFrame = part.CFrame * CFrame.new(0, 5, 0)

    -- Rotate a part 90 degrees on its own Y-axis (to make it turn)
    part.CFrame = part.CFrame * CFrame.Angles(0, math.rad(90), 0) -- Angles must be in radians
    ```

*   **Making a Part Look At Another**: `CFrame.lookAt()` is an extremely useful constructor for this.
    ```lua
    local turret = workspace.Turret
    local target = workspace.Target

    -- Creates a new CFrame located at the turret's position, but rotated to face the target's position
    turret.CFrame = CFrame.lookAt(turret.Position, target.Position)
    ```

---

