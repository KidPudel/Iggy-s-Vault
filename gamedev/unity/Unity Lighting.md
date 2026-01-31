# Unity Lighting

> [!info] Document Purpose Technical reference for Unity lighting types, modes, and techniques.

---

## 1. Overview

Lighting in Unity creates illumination and shadows that give scenes depth and atmosphere. Understanding light types and modes is essential for achieving the desired visual quality and performance balance.

---

## 2. Light Types

### 2.1 Comparison

|Type|Direction|Falloff|Shadows|Use Case|
|---|---|---|---|---|
|**Directional**|Parallel rays|None|Yes|Sun, moon|
|**Point**|All directions|Inverse square|Yes|Bulbs, torches|
|**Spot**|Cone|Inverse square|Yes|Flashlights, stage lights|
|**Area**|Rectangular surface|Inverse square|Baked only|Soft lighting, windows|

### 2.2 Directional Light

Simulates distant light sources (sun, moon). Light rays are parallel and affect the entire scene uniformly.

|Property|Description|
|---|---|
|**Color**|Light color|
|**Intensity**|Brightness|
|**Indirect Multiplier**|Bounce light intensity|
|**Shadow Type**|None, Hard, Soft|
|**Cookie**|Projected texture mask|

**Key Characteristics:**

- Position doesn't matter, only rotation
- No falloff with distance
- Linked to skybox in default scenes
- Cheapest real-time light type

### 2.3 Point Light

Emits light from a single point in all directions equally, like a light bulb.

|Property|Description|
|---|---|
|**Range**|Maximum distance light reaches|
|**Intensity**|Brightness at source|
|**Shadow Type**|None, Hard, Soft|

**Key Characteristics:**

- Intensity follows inverse square law
- Shadows require 6 shadow maps (expensive)
- Can cause light leaks through thin geometry
- Use sparingly in real-time

### 2.4 Spot Light

Emits light in a cone from a single point, like a flashlight or stage light.

|Property|Description|
|---|---|
|**Range**|Maximum distance|
|**Spot Angle**|Cone width in degrees|
|**Inner Spot Angle**|Bright center cone (HDRP)|
|**Intensity**|Brightness|
|**Cookie**|Projected texture|

**Key Characteristics:**

- Culls objects outside cone (more efficient than point)
- Penumbra (soft edge) at cone boundary
- Good for dramatic/focused lighting

### 2.5 Area Light (Baked Only)

Emits light from a rectangular or disc surface. Creates soft, realistic shadows.

|Property|Description|
|---|---|
|**Shape**|Rectangle or Disc|
|**Width/Height**|Light surface dimensions|
|**Intensity**|Brightness|

**Key Characteristics:**

- Must be baked (not real-time in Built-in/URP)
- HDRP supports real-time area lights
- Best for realistic soft lighting
- Illuminates from multiple directions

> [!note] HDRP Real-Time Area Lights HDRP supports real-time area lights with proper soft shadows, but they're expensive.

---

## 3. Light Modes

### 3.1 Mode Comparison

|Mode|Direct Light|Indirect Light|Shadows|Performance|
|---|---|---|---|---|
|**Realtime**|Realtime|None/Enlighten|Realtime|Expensive|
|**Mixed**|Realtime|Baked|Mixed|Medium|
|**Baked**|Baked|Baked|Baked|Cheap (runtime)|

### 3.2 Realtime Lighting

All lighting calculated every frame at runtime.

**Pros:**

- Dynamic lights (flickering, moving)
- Works with moving objects
- No bake time required

**Cons:**

- Performance intensive
- No global illumination (by default)
- Hard shadows only (without extra setup)

**Use for:**

- Dynamic/movable lights
- Real-time gameplay effects
- Development/prototyping

### 3.3 Baked Lighting

Lighting pre-calculated and stored in lightmaps during editor bake.

**Pros:**

- Excellent performance at runtime
- Global illumination included
- Soft shadows

**Cons:**

- Static only (can't move)
- Bake time required
- Increased storage (lightmap textures)

**Use for:**

- Static environment lighting
- Mobile games
- Architectural visualization

### 3.4 Mixed Lighting

Combines baked indirect lighting with real-time direct lighting.

**Lighting Modes:**

|Submode|Description|
|---|---|
|**Baked Indirect**|Baked GI, realtime direct + shadows|
|**Shadowmask**|Baked shadows for static, realtime for dynamic|
|**Subtractive**|Baked lighting, realtime shadows only (mobile)|

**Use for:**

- Static scenes with dynamic characters
- Games needing GI with movable objects

---

## 4. Global Illumination

Light bouncing between surfaces, creating realistic indirect lighting.

### 4.1 Baked Global Illumination

Pre-calculated during lightmap baking.

**Lightmappers:**

|Lightmapper|Description|
|---|---|
|**Progressive CPU**|Default, good quality|
|**Progressive GPU**|Faster with GPU acceleration|
|**Enlighten (deprecated)**|Legacy option|

### 4.2 Enlighten Realtime GI (Built-in RP)

Real-time indirect lighting (deprecated but still available).

### 4.3 Light Probes

Capture baked lighting for dynamic objects.

```
Setup:
1. Create → Light → Light Probe Group
2. Position probes throughout scene
3. Bake lighting
4. Dynamic objects sample nearest probes
```

**Placement Tips:**

- Denser in areas with lighting variation
- At different heights for vertical changes
- Avoid placing inside geometry

### 4.4 Reflection Probes

Capture environment reflections for reflective materials.

|Type|Description|
|---|---|
|**Baked**|Static, captured once|
|**Realtime**|Updates every frame (expensive)|
|**Custom**|Use custom cubemap|

---

## 5. Shadows

### 5.1 Shadow Types

|Type|Description|
|---|---|
|**No Shadows**|Light casts no shadows|
|**Hard Shadows**|Sharp, aliased edges|
|**Soft Shadows**|Filtered, softer edges|

### 5.2 Shadow Settings

|Property|Description|
|---|---|
|**Resolution**|Shadow map resolution (higher = sharper)|
|**Bias**|Offset to prevent shadow acne|
|**Normal Bias**|Additional offset along normals|
|**Near Plane**|Shadow frustum near plane|

### 5.3 Shadow Cascades (Directional)

Multiple shadow maps at different distances for better quality distribution.

|Cascades|Use Case|
|---|---|
|**1**|Mobile, simple scenes|
|**2**|Balanced quality/performance|
|**4**|High quality, large scenes|

### 5.4 Shadow Distance

Maximum distance from camera where shadows render.

> [!tip] Performance Reduce shadow distance for better performance. Objects beyond this distance cast no shadows.

---

## 6. Lightmapping

### 6.1 Setup

1. Mark static objects: Inspector → Static → Contribute GI
2. Open Lighting window: Window → Rendering → Lighting
3. Configure settings
4. Click "Generate Lighting" or enable Auto Generate

### 6.2 Key Settings

|Setting|Description|
|---|---|
|**Lightmap Resolution**|Texels per unit (higher = more detail)|
|**Lightmap Padding**|Space between UV islands|
|**Max Lightmap Size**|Maximum texture dimensions|
|**Compress Lightmaps**|Reduce storage size|
|**Ambient Occlusion**|Baked contact shadows|
|**Final Gather**|Improved indirect quality|

### 6.3 Lightmap UVs

Objects need proper UV2 (lightmap UVs) for baking.

```
Options:
1. Generate Lightmap UVs (Model Import Settings)
2. Use existing UV2 from modeling software
```

---

## 7. Emissive Materials

Materials can emit light and contribute to global illumination.

### 7.1 Setup

```
1. Create material
2. Enable Emission
3. Set emission color (HDR for bright glow)
4. Set Global Illumination: Baked or Realtime
5. Bake lighting
```

### 7.2 Properties

|Property|Description|
|---|---|
|**Emission Color**|HDR color of emitted light|
|**Global Illumination**|None, Realtime, Baked|

> [!note] Dynamic Objects Emissive materials only affect static geometry directly. Use Light Probes for dynamic objects to receive emissive lighting.

---

## 8. Environment Lighting

### 8.1 Ambient Light

Base lighting applied to all objects.

|Source|Description|
|---|---|
|**Skybox**|Colors from current skybox|
|**Gradient**|Custom sky/equator/ground colors|
|**Color**|Single ambient color|

### 8.2 Skybox

HDR environment that provides ambient lighting and background.

|Type|Description|
|---|---|
|**6 Sided**|Six separate textures|
|**Cubemap**|Single cubemap texture|
|**Panoramic**|Single equirectangular image|
|**Procedural**|Generated sky with sun|

### 8.3 Environment Reflections

|Source|Description|
|---|---|
|**Skybox**|Use skybox for reflections|
|**Custom**|Use specific cubemap|

---

## 9. Scripting

### 9.1 Light Component

```csharp
Light light = GetComponent<Light>();

// Properties
light.type = LightType.Point;
light.color = Color.white;
light.intensity = 1f;
light.range = 10f;           // Point/Spot only
light.spotAngle = 30f;       // Spot only
light.shadows = LightShadows.Soft;

// Enable/disable
light.enabled = true;
```

### 9.2 Dynamic Light Effects

```csharp
// Flickering light
public class FlickerLight : MonoBehaviour
{
    public Light targetLight;
    public float minIntensity = 0.5f;
    public float maxIntensity = 1.5f;
    public float flickerSpeed = 0.1f;
    
    void Update()
    {
        targetLight.intensity = Mathf.Lerp(
            minIntensity, 
            maxIntensity, 
            Mathf.PerlinNoise(Time.time * flickerSpeed, 0f)
        );
    }
}
```

### 9.3 Baking at Runtime

```csharp
// Trigger lightmap bake (Editor only)
#if UNITY_EDITOR
Lightmapping.Bake();
#endif
```

---

## 10. Pipeline Differences

### 10.1 Built-in Render Pipeline

- Forward and Deferred rendering paths
- Enlighten Realtime GI available
- Surface shaders for lighting

### 10.2 URP (Universal Render Pipeline)

- Forward rendering only
- Main Light + Additional Lights
- Light Layers for selective lighting
- No Enlighten GI

### 10.3 HDRP (High Definition Render Pipeline)

- Real-time area lights
- Volumetric lighting/fog
- Screen-space global illumination
- Ray-traced lighting (with hardware)
- Physical light units

---

## 11. Performance Tips

### 11.1 General Guidelines

|Tip|Description|
|---|---|
|**Limit real-time lights**|1 directional + few point/spot|
|**Bake when possible**|Static lights should be baked|
|**Reduce shadow quality**|Lower resolution, fewer cascades|
|**Use light layers**|Don't light everything with everything|
|**Cull by distance**|Use culling masks and ranges|

### 11.2 Mobile-Specific

|Tip|Description|
|---|---|
|**Baked only**|Minimize real-time lights|
|**No soft shadows**|Use hard shadows or none|
|**Single directional**|One real-time light maximum|
|**Low lightmap resolution**|10-20 texels per unit|
|**Compress lightmaps**|Reduce memory footprint|

### 11.3 Light Cost Comparison

|Light Type|Relative Cost|
|---|---|
|Directional (no shadows)|Low|
|Directional (shadows)|Medium|
|Spot (no shadows)|Low-Medium|
|Spot (shadows)|Medium|
|Point (no shadows)|Medium|
|Point (shadows)|High (6 shadow maps)|
|Area (baked)|Free at runtime|
|Area (real-time HDRP)|Very High|

---

## Related Links

- [[Unity Render Texture & Scriptable Render Pipeline Reference]]
- [[Unity Camera Component]]
- [[Unity Shaders]]
- [[Unity Post-Processing]]

---

> [!quote] References
> 
> - Unity Documentation: Light Types
> - Unity Documentation: Global Illumination
> - Unity Learn: Lighting Overview
> - Catlike Coding: Baked Lighting