# Unity Camera Component

> [!info] Document Purpose Technical reference for the Unity Camera component - properties, modes, and scripting.

---

## 1. Overview

The **Camera** component captures and displays the world to the player. It creates an image of a particular viewpoint in the scene, which is either drawn to the screen or captured as a texture (RenderTexture).

You can have unlimited cameras in a scene, set to render in any order, at any place on the screen, or only certain parts of the screen.

---

## 2. Projection Settings

### 2.1 Projection Types

|Type|Description|
|---|---|
|**Perspective**|Renders with depth perception. Objects farther away appear smaller. Default for 3D.|
|**Orthographic**|Renders without perspective. Objects appear same size regardless of distance. Used for 2D and UI.|
|**Physical Camera**|Simulates real-world camera behavior with sensor size, focal length, lens shift.|

### 2.2 Perspective Camera Properties

|Property|Description|
|---|---|
|**Field of View (FOV)**|View angle in degrees. Default: 60°. Higher = wider view, more distortion.|
|**FOV Axis**|Vertical (default) or Horizontal measurement axis.|
|**Near Clip Plane**|Closest distance that will be rendered. Default: 0.3.|
|**Far Clip Plane**|Furthest distance that will be rendered. Default: 1000.|

> [!tip] FOV Guidelines
> 
> - 60-75°: Standard gameplay
> - 90°+: Wide-angle, may cause motion sickness
> - 30-45°: Zoomed/sniper effect
> - Lower FOV = more "zoomed in" appearance

### 2.3 Orthographic Camera Properties

|Property|Description|
|---|---|
|**Size**|Half the vertical size of the viewing volume. Objects outside this area are clipped.|

**Calculating Orthographic Size:**

```
Size = (Screen Height / 2) / Pixels Per Unit
```

Example: 1080p with 16 PPU sprites → Size = (1080/2) / 16 = 33.75

### 2.4 Physical Camera Properties

|Property|Description|
|---|---|
|**Focal Length**|Distance from lens to sensor (mm). Determines magnification.|
|**Sensor Type**|Preset sensor sizes (35mm, Super 16, etc.)|
|**Sensor Size**|Custom width and height in millimeters.|
|**Lens Shift**|Offset of lens from center. Used for perspective correction.|
|**Gate Fit**|How sensor gate fits to resolution gate.|

---

## 3. Clipping Planes & Frustum

The **view frustum** is the pyramid-shaped region that defines what the camera can see, bounded by:

- Near clip plane (front)
- Far clip plane (back)
- Four side planes (defined by FOV)

### 3.1 Frustum Culling

Unity automatically excludes objects completely outside the frustum from rendering. This happens regardless of other culling settings.

### 3.2 Depth Buffer Precision

The depth buffer distributes precision between near and far planes. For better depth precision:

- Move near plane as far from camera as possible
- Keep the ratio of far/near as small as practical

> [!warning] Z-Fighting If surfaces close together flicker, increase the Near Clip Plane value.

---

## 4. Clear Flags

Determines what to display in empty portions of the view.

|Flag|Description|
|---|---|
|**Skybox**|Display the camera's skybox (default). Falls back to Background Color if none set.|
|**Solid Color**|Fill empty areas with Background Color.|
|**Depth Only**|Clear depth buffer only. Useful for drawing weapons over environment.|
|**Don't Clear**|Don't clear anything. Creates smear effect. Rarely used.|

### Multi-Camera Setup Example

```
Camera 1 (Environment): Depth 0, Clear Flags = Skybox
Camera 2 (Weapon):      Depth 1, Clear Flags = Depth Only
```

---

## 5. Culling

### 5.1 Culling Mask

Selectively render objects based on layers. Each layer bit in the mask determines visibility.

```csharp
// Show only "Player" layer
camera.cullingMask = 1 << LayerMask.NameToLayer("Player");

// Show everything except "UI" layer
camera.cullingMask = ~(1 << LayerMask.NameToLayer("UI"));

// Add a layer to current mask
camera.cullingMask |= 1 << LayerMask.NameToLayer("Enemy");

// Remove a layer from current mask
camera.cullingMask &= ~(1 << LayerMask.NameToLayer("Enemy"));
```

### 5.2 Per-Layer Cull Distances

Cull objects in specific layers at shorter distances for performance.

```csharp
Camera camera = GetComponent<Camera>();
float[] distances = new float[32];

// Cull "SmallDebris" layer at 50 units instead of far plane
distances[LayerMask.NameToLayer("SmallDebris")] = 50f;

camera.layerCullDistances = distances;
camera.layerCullSpherical = true; // Use sphere instead of plane
```

### 5.3 Occlusion Culling

Excludes objects hidden behind other objects. Requires baking in Unity Editor.

```csharp
camera.useOcclusionCulling = true; // Enable (default)
```

---

## 6. Rendering

### 6.1 Rendering Path (Built-in RP)

|Path|Description|
|---|---|
|**Forward**|Renders each object in passes based on lights affecting it. Best for mobile/simple scenes.|
|**Deferred**|Renders geometry info to buffers, then calculates lighting. Better for many lights.|
|**Legacy Vertex Lit**|Per-vertex lighting. Lowest quality, best performance.|
|**Legacy Deferred**|Older deferred path. Use standard Deferred instead.|

### 6.2 Target Texture

Output to a RenderTexture instead of the screen.

```csharp
RenderTexture rt = new RenderTexture(1024, 1024, 24);
camera.targetTexture = rt;
```

> [!note] Limitation A camera cannot render to both screen and RenderTexture simultaneously.

### 6.3 Viewport Rect

Define which portion of the screen the camera renders to.

|Property|Description|
|---|---|
|**X**|Left edge position (0-1)|
|**Y**|Bottom edge position (0-1)|
|**W**|Width (0-1)|
|**H**|Height (0-1)|

```csharp
// Split screen - left half
camera.rect = new Rect(0, 0, 0.5f, 1);

// Picture-in-picture - bottom right corner
camera.rect = new Rect(0.7f, 0, 0.3f, 0.3f);
```

---

## 7. Multi-Camera Setup

### 7.1 Depth (Priority)

Cameras render in order from lowest to highest depth. Higher depth cameras draw on top.

```csharp
mainCamera.depth = 0;
uiCamera.depth = 1;    // Renders after/on top of main
```

### 7.2 Common Multi-Camera Patterns

**Split-Screen Multiplayer:**

```csharp
player1Camera.rect = new Rect(0, 0.5f, 1, 0.5f);    // Top half
player2Camera.rect = new Rect(0, 0, 1, 0.5f);       // Bottom half
```

**Weapon/HUD Overlay:**

```csharp
worldCamera.depth = 0;
worldCamera.clearFlags = CameraClearFlags.Skybox;
worldCamera.cullingMask = ~(1 << LayerMask.NameToLayer("Weapon"));

weaponCamera.depth = 1;
weaponCamera.clearFlags = CameraClearFlags.Depth;
weaponCamera.cullingMask = 1 << LayerMask.NameToLayer("Weapon");
```

---

## 8. Scripting Reference

### 8.1 Common Properties

```csharp
Camera cam = Camera.main; // Main camera (tagged "MainCamera")

// Projection
cam.orthographic = false;
cam.fieldOfView = 60f;
cam.orthographicSize = 5f;
cam.nearClipPlane = 0.3f;
cam.farClipPlane = 1000f;

// Rendering
cam.backgroundColor = Color.black;
cam.clearFlags = CameraClearFlags.Skybox;
cam.cullingMask = -1; // Everything
cam.depth = 0;

// Output
cam.targetTexture = null; // Screen
cam.rect = new Rect(0, 0, 1, 1); // Full screen
```

### 8.2 Useful Methods

```csharp
// World to screen position
Vector3 screenPos = cam.WorldToScreenPoint(worldPosition);

// Screen to world (for raycasting)
Ray ray = cam.ScreenPointToRay(Input.mousePosition);

// Viewport conversions
Vector3 viewportPos = cam.WorldToViewportPoint(worldPosition);
Vector3 worldPos = cam.ViewportToWorldPoint(new Vector3(0.5f, 0.5f, 10f));

// Manual rendering
cam.Render(); // Render immediately

// Reset matrices
cam.ResetProjectionMatrix();
cam.ResetWorldToCameraMatrix();
```

### 8.3 Camera Events

```csharp
void OnPreCull() { }      // Before culling
void OnPreRender() { }    // Before rendering
void OnPostRender() { }   // After rendering
void OnRenderImage(RenderTexture src, RenderTexture dest) { } // Post-processing
```

---

## 9. Performance Tips

|Tip|Description|
|---|---|
|**Minimize cameras**|Each camera has culling/rendering overhead|
|**Reduce far clip plane**|Smaller frustum = fewer objects to process|
|**Use layer culling**|Cull small objects at shorter distances|
|**Enable occlusion culling**|For complex scenes with lots of occlusion|
|**Match target resolution**|Don't render at higher resolution than needed|
|**Disable unused features**|Turn off HDR, MSAA if not needed|

---

## 10. Pipeline-Specific Notes

### Built-in Render Pipeline

- Full control over rendering path
- Legacy post-processing stack

### URP (Universal Render Pipeline)

- Additional camera data component
- Camera stacking for overlays
- Integrated post-processing

### HDRP (High Definition Render Pipeline)

- Physical camera properties expanded
- Frame settings per camera
- Advanced anti-aliasing options

---

## Related Links

- [[Unity Render Texture & Scriptable Render Pipeline Reference]]
- [[Unity Shaders]]
- [[Unity Post-Processing]]
- [[Unity Lighting]]

---

> [!quote] References
> 
> - Unity Documentation: Camera Component
> - Unity Learn: Working with Unity Cameras
> - Unity Manual: Occlusion Culling