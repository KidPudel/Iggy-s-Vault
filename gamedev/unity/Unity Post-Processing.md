# Unity Post-Processing

> [!info] Document Purpose Technical reference for Unity post-processing effects, setup, and configuration.

---

## 1. Overview

**Post-processing** applies visual effects to the rendered image after scene rendering but before display. These full-screen filters and effects transform raw visuals into more polished, cinematic results.

Common effects include: bloom, color grading, depth of field, motion blur, ambient occlusion, and anti-aliasing.

---

## 2. Setup

### 2.1 Installation

**URP/HDRP:** Post-processing is integrated. No separate package needed.

**Built-in RP:**

1. Window → Package Manager
2. Search "Post Processing"
3. Install package

### 2.2 Basic Setup
1. **Create Post-Process Volume:**
    - GameObject → Volume → Global Volume (or Box Volume for local)
2. **Create Profile:**
    - In Volume component, click "New" next to Profile
    - Or: Project → Create → Volume Profile
3. **Add Effects:**
    - Click "Add Override" in Volume Inspector
    - Select desired effect
4. **Configure Camera (Built-in RP):**
    - Add Post-Process Layer component to camera
    - Set Layer to match Volume's layer

---

## 3. Volume System

### 3.1 Volume Types

|Type|Description|
|---|---|
|**Global**|Affects entire scene regardless of camera position|
|**Local (Box/Sphere)**|Affects only within defined boundaries|
|**Blend Distance**|Smooth transition when entering/exiting local volume|

### 3.2 Volume Priority

Higher priority volumes override lower ones. Use for area-specific effects (e.g., underwater color grading).

```
Global Volume (Priority 0) - Base settings
Local Volume (Priority 1) - Overrides when camera inside
```

### 3.3 Volume Profiles

Profiles store effect settings. Can be shared across multiple volumes.

```csharp
// Modify at runtime
Volume volume = GetComponent<Volume>();
if (volume.profile.TryGet<Bloom>(out var bloom))
{
    bloom.intensity.value = 2f;
}
```

---

## 4. Common Effects

### 4.1 Bloom

Creates glow around bright areas, simulating camera lens artifacts.

|Property|Description|
|---|---|
|**Threshold**|Minimum brightness to trigger bloom|
|**Intensity**|Strength of bloom effect|
|**Scatter**|How much bloom spreads|
|**Tint**|Color tint for bloom|
|**Clamp**|Maximum brightness in bloom|
|**High Quality Filtering**|Better quality, more expensive|
|**Lens Dirt**|Texture overlay for dirt/smudges|

**Recommended Settings:**

```
Subtle glow:     Threshold: 0.9, Intensity: 0.5-1
Dreamy look:     Threshold: 0.5, Intensity: 2-4
Sci-fi/neon:     Threshold: 0.8, Intensity: 3-5, Tint: colored
```

### 4.2 Color Grading

Adjusts colors and luminance for mood and style.

#### Tonemapping

|Mode|Description|
|---|---|
|**None**|No tonemapping (LDR only)|
|**Neutral**|Minimal hue/saturation shift|
|**ACES**|Filmic look, more contrast|

#### Basic Adjustments

|Property|Description|
|---|---|
|**Post Exposure**|Overall exposure adjustment (EV)|
|**Contrast**|Expands/compresses tonal range|
|**Color Filter**|Multiplies final color|
|**Hue Shift**|Shifts all hues|
|**Saturation**|Color intensity (-100 to 100)|

#### Trackballs (Lift/Gamma/Gain)

|Trackball|Affects|
|---|---|
|**Lift**|Shadows (dark areas)|
|**Gamma**|Midtones|
|**Gain**|Highlights (bright areas)|

#### White Balance

|Property|Description|
|---|---|
|**Temperature**|Warm (positive) to cool (negative)|
|**Tint**|Green to magenta compensation|

#### Grading Curves

- **Master:** Overall RGB intensity
- **Red/Green/Blue:** Individual channel curves
- **Hue vs Hue:** Shift specific hues to others
- **Hue vs Sat:** Adjust saturation of specific hues
- **Sat vs Sat:** Boost/reduce already saturated colors
- **Lum vs Sat:** Adjust saturation based on brightness

### 4.3 Depth of Field

Simulates camera lens focus, blurring areas outside focal range.

|Mode|Description|
|---|---|
|**Off**|Disabled|
|**Gaussian**|Simple blur, better performance|
|**Bokeh**|Realistic lens blur with bokeh shapes|

|Property|Description|
|---|---|
|**Focus Distance**|Distance at which objects are sharp|
|**Aperture**|f-stop value (lower = more blur)|
|**Focal Length**|Lens focal length in mm|
|**Blade Count**|Number of aperture blades (bokeh shape)|

### 4.4 Motion Blur

Simulates blur from camera or object movement.

|Property|Description|
|---|---|
|**Intensity**|Blur strength|
|**Clamp**|Maximum blur velocity|
|**Sample Count**|Quality (higher = better but slower)|

> [!warning] VR Disable motion blur in VR applications to prevent motion sickness.

### 4.5 Ambient Occlusion

Darkens creases, holes, and areas where surfaces meet.

|Property|Description|
|---|---|
|**Intensity**|Darkness strength|
|**Radius**|Size of occlusion sampling|
|**Direct Lighting Strength**|AO effect on direct light|
|**Quality**|Sample count for accuracy|

**Types:**

- **SSAO:** Screen-space, works everywhere
- **HBAO:** Horizon-based, more accurate
- **GTAO:** Ground-truth, best quality

### 4.6 Vignette

Darkens edges of screen, draws focus to center.

|Property|Description|
|---|---|
|**Color**|Vignette color (usually black)|
|**Center**|Vignette center point|
|**Intensity**|Darkness strength|
|**Smoothness**|Falloff softness|
|**Rounded**|Circular vs. screen-shaped|

### 4.7 Chromatic Aberration

Simulates color fringing from lens imperfections.

|Property|Description|
|---|---|
|**Intensity**|Effect strength|
|**Spectral Lut**|Custom color separation pattern|

> [!tip] Use Sparingly High chromatic aberration can be distracting. Subtle amounts add realism.

### 4.8 Film Grain

Adds noise texture simulating film stock.

|Property|Description|
|---|---|
|**Type**|Thin or Medium grain|
|**Intensity**|Visibility of grain|
|**Response**|Sensitivity to scene luminance|

### 4.9 Lens Distortion

Simulates barrel or pincushion lens distortion.

|Property|Description|
|---|---|
|**Intensity**|Positive = barrel, negative = pincushion|
|**X/Y Multiplier**|Per-axis control|
|**Center**|Distortion center point|
|**Scale**|Zoom to hide edge artifacts|

---

## 5. Anti-Aliasing

Reduces jagged edges on geometry.

### 5.1 Types Comparison

|Type|Quality|Performance|Notes|
|---|---|---|---|
|**FXAA**|Low|Fast|Slight blur, good for mobile|
|**SMAA**|Medium|Medium|Better edges than FXAA|
|**MSAA**|High|Expensive|Hardware-based, no texture aliasing|
|**TAA**|High|Medium|Uses temporal data, can ghost|

### 5.2 TAA Settings

|Property|Description|
|---|---|
|**Sharpness**|Counteract TAA blur|
|**Jitter Spread**|Sample spread amount|
|**Stationary Blending**|Blend factor for still pixels|
|**Motion Blending**|Blend factor for moving pixels|

---

## 6. URP Post-Processing

### 6.1 Camera Stack

URP uses camera stacking for overlay effects.

```
Base Camera (renders scene + post-processing)
    └── Overlay Camera (renders on top, e.g., UI)
```

### 6.2 Renderer Features

Add custom render passes for additional effects.

```csharp
public class CustomPostProcessPass : ScriptableRenderPass
{
    public override void Execute(ScriptableRenderContext context, 
                                  ref RenderingData renderingData)
    {
        // Custom post-processing logic
    }
}
```

---

## 7. HDRP Post-Processing

### 7.1 Additional Effects

HDRP includes advanced effects:

|Effect|Description|
|---|---|
|**Exposure**|Auto-exposure with adaptation|
|**Volumetric Fog**|3D atmospheric scattering|
|**Screen Space Reflection**|Real-time reflections|
|**Screen Space Global Illumination**|Real-time GI|
|**Ray-traced effects**|Hardware ray tracing|

### 7.2 Custom Pass Volume

HDRP-specific custom rendering injection points.

---

## 8. Scripting

### 8.1 Accessing Effects at Runtime

```csharp
using UnityEngine;
using UnityEngine.Rendering;
using UnityEngine.Rendering.Universal; // or HighDefinition

public class PostProcessController : MonoBehaviour
{
    public Volume volume;
    private Bloom bloom;
    private Vignette vignette;
    
    void Start()
    {
        volume.profile.TryGet(out bloom);
        volume.profile.TryGet(out vignette);
    }
    
    public void SetBloomIntensity(float intensity)
    {
        if (bloom != null)
        {
            bloom.intensity.value = intensity;
        }
    }
    
    public void EnableVignette(bool enabled)
    {
        if (vignette != null)
        {
            vignette.active = enabled;
        }
    }
}
```

### 8.2 Lerping Between Profiles

```csharp
public void TransitionToProfile(VolumeProfile targetProfile, float duration)
{
    StartCoroutine(LerpVolume(targetProfile, duration));
}

IEnumerator LerpVolume(VolumeProfile target, float duration)
{
    float elapsed = 0f;
    VolumeProfile original = volume.profile;
    
    while (elapsed < duration)
    {
        elapsed += Time.deltaTime;
        volume.weight = elapsed / duration;
        yield return null;
    }
    
    volume.profile = target;
    volume.weight = 1f;
}
```

### 8.3 Custom Effects (URP)

```csharp
using UnityEngine.Rendering;
using UnityEngine.Rendering.Universal;

[System.Serializable, VolumeComponentMenu("Custom/My Effect")]
public class MyCustomEffect : VolumeComponent, IPostProcessComponent
{
    public ClampedFloatParameter intensity = new ClampedFloatParameter(0f, 0f, 1f);
    
    public bool IsActive() => intensity.value > 0f;
    public bool IsTileCompatible() => false;
}
```

---

## 9. Performance

### 9.1 Cost by Effect

|Effect|Cost|Notes|
|---|---|---|
|Bloom|Medium|Scale down for mobile|
|Color Grading|Low|Minimal impact|
|Depth of Field|High|Use Gaussian for mobile|
|Motion Blur|Medium-High|Disable on low-end|
|Ambient Occlusion|High|Reduce quality on mobile|
|Vignette|Very Low|Always safe to use|
|Chromatic Aberration|Low|Minimal impact|
|Anti-aliasing (TAA)|Medium|Consider FXAA for mobile|

### 9.2 Optimization Tips

|Tip|Description|
|---|---|
|**Reduce resolution**|Render effects at half resolution|
|**Limit effects**|Use only necessary effects|
|**Quality settings**|Use lower quality presets on mobile|
|**Profile on target**|Test on actual hardware|
|**Disable per-platform**|Different profiles for different devices|

---

## 10. Best Practices

### 10.1 Artistic Guidelines

|Goal|Approach|
|---|---|
|**Realism**|Subtle bloom, mild color grading, soft vignette|
|**Cinematic**|ACES tonemapping, letterbox, film grain|
|**Stylized**|Strong color filter, high saturation/contrast|
|**Horror**|Desaturation, vignette, chromatic aberration|
|**Sci-fi**|Bloom on emissives, cyan/magenta color split|

### 10.2 Common Mistakes

|Mistake|Solution|
|---|---|
|Over-blooming|Keep intensity subtle (0.5-1.5)|
|Muddy colors|Don't oversaturate + lift shadows too much|
|Performance issues|Profile and disable expensive effects|
|VR sickness|Disable motion blur, chromatic aberration|
|Inconsistent look|Use profiles to maintain consistency|

---

## Related Links

- [[Unity Render Texture & Scriptable Render Pipeline Reference]]
- [[Unity Camera Component]]
- [[Unity Shaders]]
- [[Unity Lighting]]

---

> [!quote] References
> 
> - Unity Documentation: Post Processing Stack
> - Unity Learn: Introduction to Post Processing
> - Catlike Coding: Post Processing Tutorial