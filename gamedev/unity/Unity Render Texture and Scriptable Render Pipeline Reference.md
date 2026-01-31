# Unity Render Texture & Scriptable Render Pipeline Reference

> [!info] Document Purpose Technical reference for Unity RenderTexture settings and Scriptable Render Pipeline (SRP) concepts.

---

## 1. Render Texture Overview

A **Render Texture** is a special texture type that Unity creates and updates at runtime. Unlike static textures, render textures continuously update each frame, capturing a camera's output for use elsewhere in the scene.

### 1.1 Common Use Cases

|Use Case|Description|
|---|---|
|**In-world screens**|Security cameras, video calls, televisions|
|**Dynamic reflections**|Mirrors, water surfaces|
|**Portal rendering**|Showing different locations/dimensions|
|**Minimap displays**|Top-down or radar-style views|
|**Post-processing**|Image-based effects (blur, glow, color grading)|
|**Dynamic shadows**|Shadow mapping techniques|
|**Projectors**|Decals, projected textures|

### 1.2 Basic Setup Workflow

1. Create Render Texture: `Assets > Create > Render Texture`
2. Create Camera: `GameObject > Camera`
3. Assign Render Texture to camera's **Target Texture** property
4. Apply Render Texture to a Material
5. Material displays camera's view in real-time

> [!warning] Content Loss Render texture contents can become "lost" on certain events (level load, screensaver, fullscreen toggle). Check with `IsCreated()` function.

---

## 2. Render Texture Inspector Settings

### 2.1 Dimension Settings

|Value|Description|
|---|---|
|**2D**|Standard 2D texture. Most common for camera renders.|
|**2D Array**|Set of 2D textures of same size. Useful for multiple views.|
|**Cube**|Cubemap (6 faces). Used for reflections, skyboxes, environment mapping.|
|**3D**|Volume texture. Used for volumetric effects (fog, clouds).|

### 2.2 Core Properties

|Property|Description|
|---|---|
|**Size**|Resolution in pixels (width × height). Higher = better quality but more memory/GPU cost.|
|**Anti-Aliasing**|MSAA samples: `None`, `2x`, `4x`, `8x`. Smooths jagged edges.|
|**Color Format**|GraphicsFormat of color buffer. Set to `None` if only depth needed.|
|**Depth Stencil Format**|Depth/stencil buffer format: `None`, `16-bit`, `24-bit`, `32-bit`. Required for proper Z-buffering.|
|**Enable Compatible Format**|Auto-converts to platform-compatible formats if selected format unsupported.|

#### Common Color Formats

|Format|Use Case|
|---|---|
|`R8G8B8A8_UNORM`|Standard 8-bit per channel, most common|
|`R16G16B16A16_UNORM`|Higher precision, reduces banding in gradients|
|`R16G16B16A16_SFLOAT`|HDR rendering, allows values > 1.0|
|`R32G32B32A32_SFLOAT`|Maximum precision HDR|
|`None`|Depth-only texture (shadow mapping)|

#### Depth Buffer Guidelines

|Bits|Use Case|
|---|---|
|**0**|No depth buffer needed (2D effects, UI)|
|**16**|Basic depth, lower memory|
|**24**|Standard depth with 8-bit stencil|
|**32**|High precision depth|

> [!tip] Depth Buffer Importance Without a depth buffer, Z-buffering won't work and all objects render regardless of occlusion. Use at least 24-bit for 3D scenes.

### 2.3 Mipmap & Advanced Settings

|Property|Description|
|---|---|
|**Mipmap**|Allocates mipmap chain. Enable when texture viewed at varying distances.|
|**Auto-generate**|Auto-generates mipmaps. Disable for manual control via `GenerateMips()`.|
|**Dynamic Scaling**|Enables dynamic resolution. Texture resizes based on camera settings.|
|**Random Write**|Allows compute shaders to write to arbitrary locations (UAV access).|
|**Shadow Sampling Mode**|Sampling technique for shadow maps. Only visible when Color = None.|

### 2.4 Sampling Settings

#### Wrap Mode

|Mode|Behavior|
|---|---|
|**Repeat**|Tiles texture in repeating pattern|
|**Clamp**|Stretches edge pixels. Prevents wrapping artifacts.|
|**Mirror**|Tiles with mirroring at boundaries|
|**Mirror Once**|Mirrors once, then clamps|
|**Per-axis**|Different modes for U and V axes|

#### Filter Mode

|Mode|Behavior|
|---|---|
|**Point**|Nearest neighbor. Pixelated look. Best for pixel art or exact values.|
|**Bilinear**|Weighted average of 4 texels. Smooth but slightly blurry when magnified.|
|**Trilinear**|Bilinear + mip level blending. Smoothest. Requires mipmaps.|

#### Aniso Level

- Range: `0-16`
- Improves quality at steep viewing angles
- Higher = better quality but more GPU cost
- Only available when Depth Stencil Format = None

### 2.5 Video Playback Settings

For video content on RenderTextures:

```
Anti-aliasing: None
Color Format: R8G8B8A8_UNORM (or R16G16B16A16_UNORM for less banding)
Depth Stencil Format: None
Mipmap: Disabled
Wrap Mode: Clamp
Filter Mode: Point
Aniso Level: 0
```

---

## 3. Scripting Reference

### 3.1 Creating RenderTexture via Code

```csharp
// Basic creation
RenderTexture rt = new RenderTexture(1024, 1024, 24, RenderTextureFormat.ARGB32);

// With descriptor for more control
RenderTextureDescriptor desc = new RenderTextureDescriptor(1024, 1024);
desc.colorFormat = RenderTextureFormat.ARGBFloat;
desc.depthBufferBits = 24;
desc.msaaSamples = 4;
RenderTexture rt = new RenderTexture(desc);

// Don't forget to create the GPU resource
rt.Create();
```

### 3.2 Key Properties

```csharp
// Set as active render target
RenderTexture.active = myRenderTexture;

// Camera target
camera.targetTexture = myRenderTexture;

// Access buffers separately
RenderBuffer colorBuffer = rt.colorBuffer;
RenderBuffer depthBuffer = rt.depthBuffer;

// Check if created
if (!rt.IsCreated()) {
    rt.Create();
}

// Generate mipmaps manually
rt.GenerateMips();

// Enable random write for compute shaders
rt.enableRandomWrite = true;
```

### 3.3 Reading Pixels from RenderTexture

```csharp
public static Texture2D GetRTPixels(RenderTexture rt)
{
    RenderTexture currentActiveRT = RenderTexture.active;
    RenderTexture.active = rt;
    
    Texture2D tex = new Texture2D(rt.width, rt.height);
    tex.ReadPixels(new Rect(0, 0, tex.width, tex.height), 0, 0);
    tex.Apply();
    
    RenderTexture.active = currentActiveRT;
    return tex;
}
```

### 3.4 Using Separate Color and Depth Buffers

```csharp
// Create depth-only texture
RenderTexture depthTexture = new RenderTexture(1024, 1024, 24, RenderTextureFormat.Depth);

// Create color texture without depth
RenderTexture colorTexture = new RenderTexture(1024, 1024, 0);

// Set camera to use separate buffers
camera.SetTargetBuffers(colorTexture.colorBuffer, depthTexture.depthBuffer);
```

---

## 4. Scriptable Render Pipeline (SRP)

### 4.1 Overview

The **Scriptable Render Pipeline** is a thin API layer allowing developers to schedule and configure rendering commands via C# scripts. It provides full control over the rendering process.

> [!note] Pre-built Pipelines Unity provides two SRP implementations:
> 
> - **Universal Render Pipeline (URP)**: All platforms, performance-focused
> - **High Definition Render Pipeline (HDRP)**: High-end platforms, quality-focused

### 4.2 Key Components

|Component|Description|
|---|---|
|**Render Pipeline Instance**|Class inheriting from `RenderPipeline`. Defines rendering via `Render()` method.|
|**Render Pipeline Asset**|ScriptableObject inheriting from `RenderPipelineAsset`. Stores config, creates instance.|
|**ScriptableRenderContext**|Interface between C# and low-level graphics. Schedules/executes commands.|

### 4.3 Rendering Process

```
1. Culling         → Filter objects by camera visibility
2. Setup Camera    → Configure matrices, FoV, clipping planes
3. Clear Buffers   → Clear color and depth
4. Draw Objects    → Execute draw calls for culled objects
5. Submit          → Send commands to graphics API
```

### 4.4 Basic Custom SRP Structure

```csharp
public class CustomRenderPipeline : RenderPipeline
{
    protected override void Render(ScriptableRenderContext context, Camera[] cameras)
    {
        foreach (Camera camera in cameras)
        {
            // Setup camera properties
            context.SetupCameraProperties(camera);
            
            // Culling
            camera.TryGetCullingParameters(out var cullingParams);
            var cullResults = context.Cull(ref cullingParams);
            
            // Clear
            var cmd = new CommandBuffer();
            cmd.ClearRenderTarget(true, true, Color.black);
            context.ExecuteCommandBuffer(cmd);
            cmd.Release();
            
            // Draw
            var drawSettings = new DrawingSettings(
                new ShaderTagId("SRPDefaultUnlit"),
                new SortingSettings(camera)
            );
            var filterSettings = FilteringSettings.defaultValue;
            context.DrawRenderers(cullResults, ref drawSettings, ref filterSettings);
            
            // Submit
            context.Submit();
        }
    }
}
```

---

## 5. URP vs HDRP Comparison

|Aspect|URP|HDRP|
|---|---|---|
|**Target Platforms**|All (mobile, VR, consoles, PC, WebGL)|High-end PCs and consoles only|
|**Visual Fidelity**|Good quality, performance-optimized|Photorealistic, AAA-quality|
|**Rendering Path**|Forward (+ optional Forward+)|Deferred (+ Forward for transparents)|
|**Ray Tracing**|Limited support|Full support|
|**Volumetric Lighting**|Basic|Advanced (fog, clouds)|
|**Global Illumination**|Baked + Light Probes|Real-time + Baked|
|**Hardware Requirements**|Low to high|Compute shader support required|
|**Complexity**|Beginner-friendly|Advanced knowledge needed|
|**Shader Library**|UniversalRP/Lit|HDRP/Lit|

### 5.1 When to Use URP

- Mobile games
- VR applications
- Cross-platform projects
- Performance-critical applications
- Stylized/non-photorealistic graphics

### 5.2 When to Use HDRP

- AAA PC/console games
- Architectural visualization
- Film/cinematic production
- Projects requiring ray tracing
- Photorealistic rendering

### 5.3 HDRP Hardware Requirements

```
Windows: DirectX 11/12, Shader Model 5.0
PlayStation: PS4+
Xbox: Xbox One+
macOS: Metal
Linux: Vulkan

Ray Tracing requires:
- NVIDIA RTX 2060 or higher
- AMD RX 6000 series or higher
- DirectX 12
```

---

## 6. SRP and RenderTextures

### 6.1 RenderTextures in URP

```csharp
// Get camera's color texture in URP
var cameraData = camera.GetUniversalAdditionalCameraData();
cameraData.cameraStack; // For camera stacking

// Custom render features can output to RenderTextures
public class CustomRenderPass : ScriptableRenderPass
{
    private RenderTargetIdentifier source;
    private RenderTargetHandle destination;
    
    public override void Execute(ScriptableRenderContext context, 
                                  ref RenderingData renderingData)
    {
        CommandBuffer cmd = CommandBufferPool.Get();
        Blit(cmd, source, destination.Identifier());
        context.ExecuteCommandBuffer(cmd);
        CommandBufferPool.Release(cmd);
    }
}
```

### 6.2 RenderTextures in HDRP

```csharp
// HDRP Custom Pass for RenderTexture output
public class CustomPass : CustomPass
{
    public RenderTexture targetTexture;
    
    protected override void Execute(CustomPassContext ctx)
    {
        CoreUtils.SetRenderTarget(ctx.cmd, targetTexture);
        // Render operations...
    }
}
```

---

## 7. Performance Considerations

### 7.1 RenderTexture Optimization

|Factor|Recommendation|
|---|---|
|**Resolution**|Use lowest acceptable. Consider dynamic scaling.|
|**Format**|Use smallest format that meets needs (8-bit vs 16-bit vs float)|
|**Anti-aliasing**|Disable if not needed. Use 2x instead of 8x when possible.|
|**Depth buffer**|Set to 0 if not needed for 3D rendering|
|**Mipmaps**|Disable for screen-sized textures or UI|
|**Update frequency**|Don't update every frame if not needed|

### 7.2 Memory Estimates

|Resolution|32-bit Color|32-bit + 24-bit Depth|
|---|---|---|
|256×256|~0.25 MB|~0.44 MB|
|512×512|~1 MB|~1.75 MB|
|1024×1024|~4 MB|~7 MB|
|2048×2048|~16 MB|~28 MB|
|4096×4096|~64 MB|~112 MB|

> [!warning] Mobile Considerations On mobile, prefer lower resolutions and simpler formats. RenderTextures are expensive on fillrate-limited devices.

---

## 8. Quick Reference

### Creating RenderTexture in Inspector

```
Right-click in Project > Create > Render Texture
```

### Common Settings Presets

**Security Camera / Mirror:**

```
Size: 512×512 or 1024×1024
Anti-aliasing: 2x
Color Format: R8G8B8A8_UNORM
Depth: 24-bit
Filter: Bilinear
```

**Shadow Map:**

```
Size: 1024×1024 to 4096×4096
Color Format: None
Depth: 16-bit or 24-bit
Filter: Point or Bilinear
Shadow Sampling Mode: CompareDepths
```

**Post-Processing Buffer:**

```
Size: Match screen resolution
Anti-aliasing: None
Color Format: R16G16B16A16_SFLOAT (HDR)
Depth: None (unless needed)
Filter: Bilinear
```

**Minimap:**

```
Size: 256×256 to 512×512
Anti-aliasing: None or 2x
Color Format: R8G8B8A8_UNORM
Depth: 16-bit
Filter: Bilinear
Wrap: Clamp
```

---

## Related Links

- [[Unity Camera Component]]
- [[Unity Shaders]]
- [[Unity Post-Processing]]
- [[Unity Lighting]]

---

> [!quote] References
> 
> - Unity Documentation: RenderTexture
> - Unity Documentation: Scriptable Render Pipeline
> - Unity Manual: URP/HDRP Feature Comparison