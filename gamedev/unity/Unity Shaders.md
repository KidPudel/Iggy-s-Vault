# Unity Shaders

> [!quote] The Core Insight You rarely write shaders. You interact with them — through Materials and C# code. Understand the Material pipeline first, shader code second.

---

## 1. The Mental Model: How Rendering Works for Gameplay Programmers

In Godot, you assign a `ShaderMaterial` to a node and set uniforms via `set_shader_parameter()`. Unity works similarly but with an extra layer.

```
Godot:
  MeshInstance3D → material (ShaderMaterial) → shader (.gdshader)
  Code: material.set_shader_parameter("my_color", Color.RED)

Unity:
  Renderer (on GameObject) → Material (.mat asset) → Shader (.shader or Shader Graph)
  Code: renderer.material.SetColor("_BaseColor", Color.red)
```

The key difference: **Material is a standalone asset in Unity.** You create it in the Project window, assign a Shader to it, configure its properties in the Inspector, then drag it onto objects. You don't typically create Materials in code — you create them in the editor, then manipulate their properties at runtime from C#.

### The Chain

```
Shader           →  "Here are the properties you CAN configure" (blueprint)
Material         →  "Here are the VALUES I've chosen" (instance of blueprint)
Renderer         →  "Here is the material ON this object" (connection to GameObject)
```

A single Shader can be used by many Materials. A single Material can be shared across many Renderers. This is important for performance — shared Materials batch together.

---

## 2. Godot → Unity Translation

### Concepts

|Godot|Unity|Notes|
|---|---|---|
|`ShaderMaterial`|`Material`|Both connect a shader to property values|
|`.gdshader` file|`.shader` file or Shader Graph asset|Unity has two authoring methods|
|`shader_type spatial`|Lit Shader Graph (or Surface/Vertex-Fragment)|3D with lighting|
|`shader_type canvas_item`|Sprite Lit/Unlit Shader Graph|2D shaders|
|`uniform float my_val`|Shader property (`_MyVal`)|Exposed to Inspector|
|Visual Shader editor|Shader Graph|Node-based, nearly identical concept|
|`material_override`|`renderer.material` (instance)|Per-object override|
|Shared material (default)|`renderer.sharedMaterial`|Shared across objects|

### Setting Parameters from Code

```gdscript
# Godot
var mat = $MeshInstance3D.material_override as ShaderMaterial
mat.set_shader_parameter("dissolve_amount", 0.5)
mat.set_shader_parameter("tint_color", Color.RED)
var val = mat.get_shader_parameter("dissolve_amount")
```

```csharp
// Unity — see Section 4 for the full API and gotchas
var renderer = GetComponent<Renderer>();
renderer.material.SetFloat("_DissolveAmount", 0.5f);
renderer.material.SetColor("_TintColor", Color.red);
float val = renderer.material.GetFloat("_DissolveAmount");
```

The concept is the same. The differences: Unity uses typed methods (`SetFloat`, `SetColor`, `SetTexture`) instead of one generic `set_shader_parameter`, and property names are prefixed with underscore by convention (`_DissolveAmount` not `dissolve_amount`).

---

## 3. Which Shader Approach to Use

### The Decision (URP — use this for new projects)

```
Do you need a custom visual effect?
├── NO → Use the built-in URP Lit or Unlit shader. Done.
│         (Covers 70%+ of gameplay needs out of the box)
│
├── SIMPLE EFFECT (tint, dissolve, glow, outline) 
│   → Shader Graph (visual node editor, no code)
│
├── COMPLEX EFFECT (custom lighting, procedural geometry)
│   → Hand-written HLSL vertex/fragment shader
│
└── NOT RENDERING (GPU simulation, compute)
    → Compute Shader (advanced, you'll know when you need it)
```

**As a gameplay programmer, your default path is:** use the built-in Lit/Unlit shaders and control them from C#. When you need a custom effect, reach for Shader Graph first — it's faster to iterate and hard to break. Write HLSL only when Shader Graph can't do what you need.

### Render Pipeline Choice

|Pipeline|Use When|Shader Authoring|
|---|---|---|
|**URP** (Universal)|New projects, mobile, cross-platform, most games|Shader Graph + HLSL|
|**HDRP** (High Definition)|AAA PC/console, photorealism|Shader Graph + HLSL|
|**Built-in**|Legacy projects only|Surface Shaders + HLSL|

**Pick URP for new projects.** It's the modern default, performs well everywhere, and has the widest community support. Surface Shaders (you'll see them in old tutorials) are Built-in RP only — don't use them with URP.

### Creating a Material (the workflow you'll use daily)

1. **Project window:** Right-click → Create → Material
2. **Inspector:** Choose shader from dropdown (e.g., `Universal Render Pipeline/Lit`)
3. **Configure:** Set properties — base color, smoothness, textures, emission, etc.
4. **Apply:** Drag material onto a GameObject, or assign to the Renderer's Material slot

That's it. No code needed for static materials.

---

## 4. C# Material API — The Gameplay Programmer's Main Tool

This is where you'll spend most of your shader-adjacent time: changing material properties at runtime.

### The Basics

```csharp
public class DamageFlash : MonoBehaviour
{
    [SerializeField] private Color _flashColor = Color.red;
    
    private Renderer _renderer;
    
    private void Awake()
    {
        _renderer = GetComponent<Renderer>();
    }
    
    public void Flash()
    {
        // Set the material's color property
        _renderer.material.SetColor("_BaseColor", _flashColor);
    }
    
    public void ResetColor()
    {
        _renderer.material.SetColor("_BaseColor", Color.white);
    }
}
```

### Typed Setter/Getter Methods

|Method|Godot Equivalent|Example|
|---|---|---|
|`material.SetFloat("_Name", value)`|`set_shader_parameter("name", float)`|`mat.SetFloat("_DissolveAmount", 0.5f)`|
|`material.SetColor("_Name", color)`|`set_shader_parameter("name", Color)`|`mat.SetColor("_BaseColor", Color.red)`|
|`material.SetTexture("_Name", tex)`|`set_shader_parameter("name", Texture)`|`mat.SetTexture("_MainTex", myTexture)`|
|`material.SetVector("_Name", vec)`|`set_shader_parameter("name", Vector)`|`mat.SetVector("_Offset", new Vector4(...))`|
|`material.SetInt("_Name", val)`|`set_shader_parameter("name", int)`|`mat.SetInt("_UseEmission", 1)`|
|`material.GetFloat("_Name")`|`get_shader_parameter("name")`|`float v = mat.GetFloat("_Cutoff")`|
|`material.GetColor("_Name")`|`get_shader_parameter("name")`|`Color c = mat.GetColor("_BaseColor")`|

### Common URP Property Names

These are the property names for the standard URP Lit shader. You need to know these:

|Property|Type|What It Controls|
|---|---|---|
|`_BaseColor`|Color|Main tint (Built-in uses `_Color`)|
|`_BaseMap`|Texture2D|Main texture (Built-in uses `_MainTex`)|
|`_Smoothness`|Float (0-1)|Surface smoothness|
|`_Metallic`|Float (0-1)|Metallic amount|
|`_BumpMap`|Texture2D|Normal map|
|`_BumpScale`|Float|Normal map intensity|
|`_EmissionColor`|Color (HDR)|Emission color and intensity|
|`_EmissionMap`|Texture2D|Emission texture|
|`_Cutoff`|Float|Alpha cutoff threshold|

> [!warning] Built-in RP uses different names: `_Color` instead of `_BaseColor`, `_MainTex` instead of `_BaseMap`. If a tutorial uses `_Color`, it's probably Built-in RP. URP uses `_BaseColor`.

### How to Find Property Names

You'll encounter shaders (from assets, teammates, Shader Graph) where you don't know the property names. Here's how to find them:

1. **Inspector:** Select the Material → property names shown in debug mode (Inspector → three-dot menu → Debug)
2. **Shader source:** Open the `.shader` file → look in the `Properties` block
3. **Shader Graph:** Open the graph → properties are in the Blackboard panel on the left
4. **Code:** `material.shader.GetPropertyName(index)` iterates all properties

---

## 5. The .material vs .sharedMaterial Trap

This is the most common material-related mistake in Unity. Godot has a similar concept (`resource_local_to_scene`) but Unity's version is more dangerous because it happens silently.

### The Problem

```csharp
// THIS CREATES A CLONE every time you access .material
renderer.material.SetColor("_BaseColor", Color.red);
// Unity internally does: "oh, you want to modify this? Let me copy it first."
// Now this renderer has its own unique Material instance.
// The original shared Material is untouched.
```

**What happened:** Accessing `.material` (getter) silently clones the material if it was shared. You now have a unique material instance on this renderer. This means:

- This object can no longer batch with others that use the original material
- If you do this on 500 enemies, you have 500 material clones in memory
- In the editor, the material shows as "MaterialName (Instance)" — a clue something went wrong

### When Each Is Appropriate

```csharp
// READ a shared property (same for all objects using this material)
Color c = renderer.sharedMaterial.GetColor("_BaseColor");  // Safe, no clone

// WRITE to all objects sharing this material (e.g., global tint)
renderer.sharedMaterial.SetColor("_BaseColor", Color.red);  // Changes ALL objects
// WARNING: In Editor, this persists after play mode ends!

// WRITE to one specific object (e.g., damage flash on one enemy)
renderer.material.SetColor("_BaseColor", Color.red);  // Clones, affects only this object
// Fine for a few objects. Bad for hundreds.
```

### The Decision

|Situation|Use|Why|
|---|---|---|
|Read any property|`.sharedMaterial`|No clone, just reading|
|Change ALL objects with this material|`.sharedMaterial`|One material, all references update|
|Change ONE object (rare, few objects)|`.material`|Creates a clone for this object only|
|Change many individual objects|`MaterialPropertyBlock`|No clones, best performance|

### MaterialPropertyBlock — The Right Way for Many Objects

When you need per-object variation (different tint per enemy, different dissolve amount) without creating material clones:

```csharp
public class EnemyTint : MonoBehaviour
{
    [SerializeField] private Color _teamColor;
    
    // Cache the property ID for performance (avoid string lookup every frame)
    private static readonly int BaseColorID = Shader.PropertyToID("_BaseColor");
    
    private Renderer _renderer;
    private MaterialPropertyBlock _propBlock;
    
    private void Awake()
    {
        _renderer = GetComponent<Renderer>();
        _propBlock = new MaterialPropertyBlock();
    }
    
    private void Start()
    {
        SetColor(_teamColor);
    }
    
    public void SetColor(Color color)
    {
        // Get existing properties (preserves anything set by other scripts)
        _renderer.GetPropertyBlock(_propBlock);
        
        // Set our property
        _propBlock.SetColor(BaseColorID, color);
        
        // Apply
        _renderer.SetPropertyBlock(_propBlock);
    }
}
```

**How it works:** The material stays shared (good for batching). The `MaterialPropertyBlock` overrides specific properties per-renderer without cloning anything. 500 enemies with different colors? One material, 500 lightweight property blocks.

> [!warning] **URP/SRP Batcher caveat:** `MaterialPropertyBlock` breaks SRP Batcher compatibility (the SRP Batcher is URP's default batching strategy). For small numbers of objects this is fine. For hundreds, consider using pre-made material variants or GPU instancing instead. Test with the Frame Debugger (`Window → Analysis → Frame Debugger`) to see actual batching behavior.

### Shader.PropertyToID — Cache Property Names

String-based property lookups are slow. If you're setting properties frequently (every frame), cache the ID:

```csharp
// Do this once (static readonly = shared across all instances)
private static readonly int DissolveID = Shader.PropertyToID("_DissolveAmount");
private static readonly int TintID = Shader.PropertyToID("_BaseColor");

// Use the cached ID instead of the string
material.SetFloat(DissolveID, dissolveValue);
material.SetColor(TintID, tintColor);
```

---

## 6. Swapping Materials at Runtime

Sometimes you don't want to change a property — you want to swap the entire material (e.g., ghost mode, stealth, highlight).

```csharp
public class MaterialSwapper : MonoBehaviour
{
    [SerializeField] private Material _normalMaterial;
    [SerializeField] private Material _ghostMaterial;
    
    private Renderer _renderer;
    
    private void Awake()
    {
        _renderer = GetComponent<Renderer>();
    }
    
    public void BecomeGhost()
    {
        // Assign a different pre-made material
        _renderer.sharedMaterial = _ghostMaterial;
    }
    
    public void BecomeNormal()
    {
        _renderer.sharedMaterial = _normalMaterial;
    }
}
```

Use `.sharedMaterial` for assignment — you're pointing to an existing asset, not cloning.

For objects with multiple materials (multi-material meshes):

```csharp
// Get all materials
Material[] mats = renderer.sharedMaterials;

// Swap the second material (index 1)
mats[1] = newMaterial;

// Assign back (must assign the whole array)
renderer.sharedMaterials = mats;
```

---

## 7. Enabling/Disabling Shader Features from C#

Some shaders have keyword-toggled features (emission, transparency, normal maps). The URP Lit shader uses these extensively.

```csharp
// Enable emission at runtime
material.EnableKeyword("_EMISSION");
material.SetColor("_EmissionColor", Color.yellow * 2f);  // HDR: multiply for intensity

// Disable emission
material.DisableKeyword("_EMISSION");

// Change surface type to transparent
material.SetFloat("_Surface", 1);  // 0 = Opaque, 1 = Transparent
material.SetFloat("_Blend", 0);    // 0 = Alpha, 1 = Premultiply, 2 = Additive, 3 = Multiply
material.SetInt("_SrcBlend", (int)UnityEngine.Rendering.BlendMode.SrcAlpha);
material.SetInt("_DstBlend", (int)UnityEngine.Rendering.BlendMode.OneMinusSrcAlpha);
material.SetInt("_ZWrite", 0);
material.renderQueue = 3000;
```

> [!tip] Switching between opaque and transparent at runtime is fiddly (as you can see above). If you need this, it's often easier to have two separate materials (one opaque, one transparent) and swap between them.

---

## 8. Common Gameplay Shader Tasks

Practical patterns you'll actually use:

### Damage Flash (Color Tint)

```csharp
public class DamageFlash : MonoBehaviour
{
    [SerializeField] private Color _flashColor = Color.white;
    [SerializeField] private float _flashDuration = 0.1f;
    
    private static readonly int BaseColorID = Shader.PropertyToID("_BaseColor");
    
    private Renderer _renderer;
    private Color _originalColor;
    private Coroutine _flashRoutine;
    
    private void Awake()
    {
        _renderer = GetComponent<Renderer>();
        _originalColor = _renderer.sharedMaterial.GetColor(BaseColorID);
    }
    
    public void Flash()
    {
        if (_flashRoutine != null) StopCoroutine(_flashRoutine);
        _flashRoutine = StartCoroutine(FlashRoutine());
    }
    
    private IEnumerator FlashRoutine()
    {
        _renderer.material.SetColor(BaseColorID, _flashColor);
        yield return new WaitForSeconds(_flashDuration);
        _renderer.material.SetColor(BaseColorID, _originalColor);
    }
}
```

### Dissolve Effect (requires custom shader/Shader Graph with `_DissolveAmount` property)

```csharp
public class DissolveOnDeath : MonoBehaviour
{
    [SerializeField] private float _dissolveDuration = 1.5f;
    
    private static readonly int DissolveID = Shader.PropertyToID("_DissolveAmount");
    
    private Renderer _renderer;
    
    public void StartDissolve()
    {
        StartCoroutine(DissolveRoutine());
    }
    
    private IEnumerator DissolveRoutine()
    {
        _renderer = GetComponent<Renderer>();
        float elapsed = 0f;
        
        while (elapsed < _dissolveDuration)
        {
            elapsed += Time.deltaTime;
            float t = elapsed / _dissolveDuration;
            _renderer.material.SetFloat(DissolveID, t);
            yield return null;
        }
        
        Destroy(gameObject);
    }
}
```

### Material Lerp (Smooth Transition Between Two Materials)

```csharp
// Smoothly blend between two materials
renderer.material.Lerp(materialA, materialB, t);  // t: 0 = A, 1 = B
```

### Texture Offset Animation (Scrolling UV)

```csharp
// Scroll a texture (e.g., conveyor belt, water flow)
private static readonly int BaseMapID = Shader.PropertyToID("_BaseMap_ST");

void Update()
{
    float offset = Time.time * _scrollSpeed;
    _renderer.material.SetTextureOffset("_BaseMap", new Vector2(offset, 0));
}
```

---

## 9. Shader Graph Basics

When the built-in Lit/Unlit shaders aren't enough and you need a custom effect, Shader Graph is your tool. It's Unity's equivalent to Godot's Visual Shader editor.

### What It Is

A node-based visual editor where you wire nodes together to create shaders. No HLSL code required. Available in URP and HDRP only (not Built-in RP).

### Creating a Shader Graph

1. Project window → Create → Shader Graph → URP → **Lit Shader Graph** (or Unlit)
2. Double-click to open the graph editor
3. Add properties in the **Blackboard** panel (left side) — these become Material Inspector fields
4. Wire nodes together on the graph surface
5. Connect final outputs to the **Fragment** output node

### Key Concepts

|Concept|Godot Visual Shader Equivalent|
|---|---|
|Blackboard properties|Uniform inputs|
|Fragment outputs|Fragment function outputs|
|Vertex outputs|Vertex function outputs|
|Sub Graphs|Reusable node groups|
|Custom Function node|Inline shader code within the graph|
|Keywords|Shader feature toggles|

### Common Properties You'll Expose

|Property Type|Use For|
|---|---|
|Float (Slider 0-1)|Dissolve amount, fade, blend|
|Color (HDR)|Tint, emission, outline color|
|Texture2D|Noise mask, pattern, albedo|
|Boolean/Keyword|Feature toggle (on/off)|
|Vector2|UV offset, tiling|

### Useful Nodes for Gameplay Effects

|Node|What It Does|Used For|
|---|---|---|
|**Time**|Current time value|Animation, scrolling, pulsing|
|**UV**|Texture coordinates|Any texture manipulation|
|**Noise (Simple/Gradient/Voronoi)**|Procedural patterns|Dissolve masks, distortion|
|**Step**|Hard threshold cutoff|Dissolve edge, toon shading|
|**Smoothstep**|Soft threshold|Gradual transitions|
|**Lerp**|Blend between two values|Color mixing, transitions|
|**Fresnel Effect**|Edge glow based on view angle|Rim lighting, shields, hologram|
|**Position** (World)|World-space position|Height-based effects|
|**Sample Texture 2D**|Read a texture|Any texture input|
|**Tiling And Offset**|Animate/scale UVs|Scrolling, scaling patterns|

### Gameplay Effects You Can Build in Shader Graph

|Effect|Key Nodes|Difficulty|
|---|---|---|
|**Color tint**|Multiply with Color property|Trivial|
|**Dissolve**|Noise + Step + Alpha Clip|Easy|
|**Scrolling texture**|Time + Tiling And Offset|Easy|
|**Rim glow / Fresnel**|Fresnel Effect + Emission|Easy|
|**Hologram**|Fresnel + Scanlines (frac of Position.y) + Emission|Medium|
|**Shield impact**|World Position + distance from hit point + Fresnel|Medium|
|**Outline**|Two-pass (vertex extrusion) or Fullscreen Shader Graph|Medium-Hard|
|**Toon/cel shading**|Custom lighting with Step thresholds|Hard (needs Custom Function node)|

---

## 10. Shader Types Reference

When you do need to understand what's under the hood:

|Type|What It Is|When You'd Touch It|
|---|---|---|
|**Shader Graph**|Visual node editor → generates shader code|Custom effects, your primary tool|
|**Unlit**|No lighting. Flat color/texture|UI, particles, skyboxes, stylized|
|**Vertex/Fragment (HLSL)**|Full control, hand-written|When Shader Graph can't do it|
|**Surface** (Built-in RP only)|Auto-handles lighting/shadows|Legacy projects only, avoid for new work|
|**Compute**|GPU parallel computation, not rendering|Particle sims, procedural gen, advanced|

### ShaderLab Structure (for reading, not writing)

You don't need to write this, but you'll encounter it in assets and documentation:

```hlsl
Shader "Category/ShaderName"        // Name in material dropdown
{
    Properties                       // What shows in Inspector
    {
        _BaseColor ("Color", Color) = (1,1,1,1)
        _MainTex ("Texture", 2D) = "white" {}
        _DissolveAmount ("Dissolve", Range(0,1)) = 0
    }
    
    SubShader                        // GPU instructions
    {
        Tags { "RenderType"="Opaque" "RenderPipeline"="UniversalPipeline" }
        
        Pass
        {
            HLSLPROGRAM
            #pragma vertex vert      // Vertex function name
            #pragma fragment frag    // Fragment function name
            
            // ... HLSL code ...
            
            ENDHLSL
        }
    }
    
    Fallback "Universal Render Pipeline/Lit"  // If GPU can't run this shader
}
```

**The Properties block** is the most relevant part for you — it defines what property names (like `_BaseColor`, `_DissolveAmount`) exist and what types they are. This is what you reference when calling `material.SetFloat("_DissolveAmount", value)` from C#.

### Properties Block Quick Reference

|Type|Syntax|Example|
|---|---|---|
|**Color**|`_Name ("Label", Color) = (r,g,b,a)`|`_BaseColor ("Color", Color) = (1,1,1,1)`|
|**Float**|`_Name ("Label", Float) = value`|`_Glossiness ("Smoothness", Float) = 0.5`|
|**Range**|`_Name ("Label", Range(min,max)) = value`|`_Metallic ("Metallic", Range(0,1)) = 0`|
|**Vector**|`_Name ("Label", Vector) = (x,y,z,w)`|`_Offset ("Offset", Vector) = (0,0,0,0)`|
|**2D Texture**|`_Name ("Label", 2D) = "default" {}`|`_MainTex ("Albedo", 2D) = "white" {}`|

---

## 11. Render States — What They Mean When You See Them

You won't set these from C# often, but you need to understand them when troubleshooting visual issues:

### Why Is My Object Transparent / Invisible / Rendering Wrong?

|Problem|Likely Cause|Check|
|---|---|---|
|Object invisible|Render Queue wrong, or shader culling|Material → Render Queue, Cull setting|
|Transparent object hides things behind it|`ZWrite On` with transparency|Should be `ZWrite Off` for transparent|
|Object renders on top of everything|Render Queue too high|Set to Geometry (2000) for opaque|
|See-through from one side|`Cull Back` (default)|Set to `Cull Off` for double-sided|
|Z-fighting (flickering overlap)|Two surfaces at same depth|Offset one, or use Render Queue offset|

### Render Queue Values

|Queue|Value|What It's For|
|---|---|---|
|Background|1000|Skyboxes|
|Geometry|2000|Opaque objects (default)|
|AlphaTest|2450|Cutout transparency|
|Transparent|3000|See-through objects|
|Overlay|4000|UI overlays, lens flares|

Opaque objects render front-to-back (fast — skips hidden pixels). Transparent objects render back-to-front (slow — must blend). This is why making everything transparent kills performance.

### Common Blend Modes (for reference)

```hlsl
Blend SrcAlpha OneMinusSrcAlpha  // Standard transparency
Blend One One                      // Additive (glow, fire, particles)
Blend DstColor Zero                // Multiplicative (darken)
```

---

## 12. Performance Tips for Gameplay Programmers

|Tip|Why|
|---|---|
|**Don't access `.material` in Update**|Creates clones every frame if not already cloned|
|**Cache `Shader.PropertyToID()`**|String lookup is slow; int lookup is fast|
|**Use `MaterialPropertyBlock` for per-object variation**|Avoids material clones|
|**Pre-make material variants**|Swap whole materials instead of toggling many properties|
|**Limit transparent objects**|Transparent = back-to-front sorting = expensive|
|**Use Frame Debugger**|`Window → Analysis → Frame Debugger` shows actual draw calls|
|**Avoid `Renderer.materials` (plural)**|Returns a COPY of the array. Every. Time.|

---

## Related Links

- [[Unity Render Texture & Scriptable Render Pipeline Reference]]
- [[Unity Camera Component]]
- [[Unity Post-Processing]]
- [[Unity Lighting]]

---

> [!quote] References
> 
> - Unity Documentation: Material API
> - Unity Documentation: Shader Graph
> - Unity Manual: ShaderLab Syntax
> - Catlike Coding: Shader Tutorials
> - Daniel Ilett: URP Shader Tutorials
> - Cyanilux: URP Shader Code