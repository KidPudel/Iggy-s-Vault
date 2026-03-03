### **1. Prepare the Mesh**
- Apply transforms: Ctrl + A → Apply **Rotation** and **Scale**.
- Set origin: Right Click → **Set Origin → Origin to Geometry** (or to base of model).
- Rename mesh clearly (e.g. CharacterMesh).

---

### **2. Add Armature (Rig)**
- Shift + A → **Armature → Single Bone**
- Rename armature (Armature, Rig, etc.)
- In **Object Data Properties → Viewport Display**: enable **In Front** (for visibility).
- Tab into **Edit Mode** to start building bones.

---

### **3. Build the Rig**
- Use E to extrude bones for:
    - Spine, Head
    - Arms (shoulder, upper, lower, hand)
    - Legs (upper, lower, foot, toe)
- Mirror bones:
    - Name left side with .L, then Armature → Armature panel → **Symmetrize**
        
- Set bone roll as needed (Ctrl + R in Edit Mode)
    

---

### **4. Weight Binding**
- Select **Mesh**, then **Armature** (Shift + Click)
- Ctrl + P → **With Automatic Weights**

  

**Optional cleanup:**

- Enter **Weight Paint Mode** to verify.
    
- Use Weights → Normalize All or manually fix bad areas.
    

---

### **✅** 

### **5. Test the Rig**

- Select Armature → Pose Mode.
    
- Select bones → R to rotate, test deformations.
    
- Add IK constraints if needed (e.g. for legs/arms).
    

---

### **✅** 

### **6. Weight Cleanup (if needed)**

- In Weight Paint or Vertex Groups:
    
    - Remove unused groups.
        
    - Manually assign or smooth weights.
        
    
- Use **Limit Total** modifier to clamp max influences.
    

---

### **✅** 

### **7. Optional: Use Rigify (Auto-Rig)**

- Enable Add-on: Edit → Preferences → Add-ons → Rigify
    
- Add → Armature → **Human (Meta-Rig)**
    
- Adjust to fit mesh.
    
- Click **Generate Rig**
    
- Re-parent mesh to generated rig (same as above).
    

---

### **✅** 

### **8. Final Tips**

- Apply modifiers **before** binding (e.g. Mirror).
    
- Keep backup of mesh before binding.
    
- Keep one Armature Modifier; others can conflict.
    
- Keep bone naming consistent (.L, .R) for auto-mirroring and constraints.
    

---