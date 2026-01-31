# Unity Shaders

> [!info] Document Purpose Technical reference for Unity shader types, structure, and writing shader code.

---

## 1. Overview

A **Shader** is a program that runs on the GPU, containing mathematical calculations and algorithms for determining the color of each pixel rendered based on lighting and material configuration.

Unity uses **ShaderLab** as a wrapper language that contains HLSL/Cg code for the actual GPU programs.

---

## 2. Shader Types

### 2.1 Comparison

|Type|Use Case|Lighting|Complexity|
|---|---|---|---|
|**Unlit**|UI, particles, special effects|No built-in|Simple|
|**Vertex/Fragment**|Custom effects, post-processing|Manual|Medium|
|**Surface**|Physically-based materials|Built-in|Abstracted|
|**Compute**|GPGPU, simulations|N/A|Advanced|

### 2.2 Unlit Shaders

No lighting calculations. Objects appear with flat color/texture regardless of scene lighting.

**Use cases:** UI elements, particles, skyboxes, fullscreen effects, stylized graphics

### 2.3 Vertex/Fragment Shaders

Low-level shaders with full control over the rendering process.

**Vertex Shader:** Processes each vertex, transforms positions, prepares data for fragment shader.

**Fragment Shader:** Processes each pixel, determines final color output.

### 2.4 Surface Shaders (Built-in RP only)

Abstracted shaders that automatically handle lighting, shadows, and multiple render paths. Surface shaders compile down to vertex/fragment shaders.

> [!warning] URP/HDRP Surface shaders are not supported in URP or HDRP. Use Shader Graph or custom vertex/fragment shaders instead.

### 2.5 Compute Shaders

General-purpose GPU programs for parallel computation. Not for rendering directly.

**Use cases:** Particle simulations, physics, image processing, procedural generation

---

## 3. ShaderLab Structure

```hlsl
Shader "Category/ShaderName"
{
    Properties
    {
        // Material properties exposed in Inspector
        _Color ("Color", Color) = (1,1,1,1)
        _MainTex ("Texture", 2D) = "white" {}
    }
    
    SubShader
    {
        Tags { "RenderType"="Opaque" "Queue"="Geometry" }
        LOD 100
        
        Pass
        {
            CGPROGRAM
            // Shader code here
            ENDCG
        }
    }
    
    Fallback "Diffuse"
}
```

### 3.1 Properties Block

|Type|Syntax|Example|
|---|---|---|
|**Color**|`_Name ("Label", Color) = (r,g,b,a)`|`_Color ("Tint", Color) = (1,1,1,1)`|
|**Float**|`_Name ("Label", Float) = value`|`_Glossiness ("Smoothness", Float) = 0.5`|
|**Range**|`_Name ("Label", Range(min,max)) = value`|`_Metallic ("Metallic", Range(0,1)) = 0`|
|**Vector**|`_Name ("Label", Vector) = (x,y,z,w)`|`_Offset ("Offset", Vector) = (0,0,0,0)`|
|**2D Texture**|`_Name ("Label", 2D) = "default" {}`|`_MainTex ("Albedo", 2D) = "white" {}`|
|**Cube**|`_Name ("Label", Cube) = "" {}`|`_Cubemap ("Reflection", Cube) = "" {}`|
|**3D**|`_Name ("Label", 3D) = "" {}`|`_Volume ("Volume", 3D) = "" {}`|

### 3.2 SubShader Tags

|Tag|Values|Description|
|---|---|---|
|**RenderType**|Opaque, Transparent, TransparentCutout|Categorizes shader for replacement|
|**Queue**|Background, Geometry, AlphaTest, Transparent, Overlay|Render order|
|**DisableBatching**|True, False|Disable batching for this shader|
|**ForceNoShadowCasting**|True|Don't cast shadows|
|**IgnoreProjector**|True|Ignore projectors|

**Queue Values:**

|Queue|Value|Use Case|
|---|---|---|
|Background|1000|Skyboxes|
|Geometry|2000|Opaque objects (default)|
|AlphaTest|2450|Alpha-tested objects|
|Transparent|3000|Transparent objects|
|Overlay|4000|Lens flares, UI|

---

## 4. Vertex/Fragment Shader

### 4.1 Basic Structure

```hlsl
Shader "Custom/BasicUnlit"
{
    Properties
    {
        _MainTex ("Texture", 2D) = "white" {}
        _Color ("Color", Color) = (1,1,1,1)
    }
    
    SubShader
    {
        Tags { "RenderType"="Opaque" }
        
        Pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            
            #include "UnityCG.cginc"
            
            // Properties as variables
            sampler2D _MainTex;
            float4 _MainTex_ST;
            float4 _Color;
            
            // Input from mesh
            struct appdata
            {
                float4 vertex : POSITION;
                float2 uv : TEXCOORD0;
            };
            
            // Output from vertex to fragment
            struct v2f
            {
                float4 vertex : SV_POSITION;
                float2 uv : TEXCOORD0;
            };
            
            // Vertex shader
            v2f vert (appdata v)
            {
                v2f o;
                o.vertex = UnityObjectToClipPos(v.vertex);
                o.uv = TRANSFORM_TEX(v.uv, _MainTex);
                return o;
            }
            
            // Fragment shader
            fixed4 frag (v2f i) : SV_Target
            {
                fixed4 col = tex2D(_MainTex, i.uv) * _Color;
                return col;
            }
            ENDCG
        }
    }
}
```

### 4.2 Semantics

**Input Semantics (appdata):**

|Semantic|Type|Description|
|---|---|---|
|`POSITION`|float4|Vertex position|
|`NORMAL`|float3|Vertex normal|
|`TANGENT`|float4|Vertex tangent|
|`TEXCOORD0-7`|float2/3/4|UV coordinates|
|`COLOR`|float4|Vertex color|

**Output Semantics:**

|Semantic|Description|
|---|---|
|`SV_POSITION`|Clip-space position (required)|
|`SV_Target`|Fragment output color|
|`SV_Depth`|Custom depth value|

### 4.3 Common Include Files

```hlsl
#include "UnityCG.cginc"        // Common functions
#include "Lighting.cginc"        // Lighting helpers
#include "AutoLight.cginc"       // Shadow helpers
#include "UnityPBSLighting.cginc" // PBS functions
```

### 4.4 Useful Functions (UnityCG.cginc)

```hlsl
// Transformations
UnityObjectToClipPos(vertex)      // Object to clip space
UnityObjectToWorldNormal(normal)  // Object to world normal
UnityWorldSpaceViewDir(worldPos)  // View direction

// Texture
TRANSFORM_TEX(uv, _MainTex)       // Apply tiling/offset

// Lighting
WorldSpaceLightDir(worldPos)      // Direction to light
Shade4PointLights(...)            // Compute 4 point lights

// Utility
DecodeHDR(data, hdr)              // Decode HDR value
LinearToGammaSpace(color)         // Color space conversion
```

---

## 5. Surface Shaders (Built-in RP)

### 5.1 Basic Structure

```hlsl
Shader "Custom/BasicSurface"
{
    Properties
    {
        _Color ("Color", Color) = (1,1,1,1)
        _MainTex ("Albedo", 2D) = "white" {}
        _Glossiness ("Smoothness", Range(0,1)) = 0.5
        _Metallic ("Metallic", Range(0,1)) = 0.0
    }
    
    SubShader
    {
        Tags { "RenderType"="Opaque" }
        
        CGPROGRAM
        #pragma surface surf Standard fullforwardshadows
        #pragma target 3.0
        
        sampler2D _MainTex;
        half _Glossiness;
        half _Metallic;
        fixed4 _Color;
        
        struct Input
        {
            float2 uv_MainTex;
        };
        
        void surf (Input IN, inout SurfaceOutputStandard o)
        {
            fixed4 c = tex2D(_MainTex, IN.uv_MainTex) * _Color;
            o.Albedo = c.rgb;
            o.Metallic = _Metallic;
            o.Smoothness = _Glossiness;
            o.Alpha = c.a;
        }
        ENDCG
    }
    
    FallBack "Diffuse"
}
```

### 5.2 Lighting Models

|Model|Description|
|---|---|
|`Standard`|Physically-based (metallic workflow)|
|`StandardSpecular`|Physically-based (specular workflow)|
|`Lambert`|Diffuse only|
|`BlinnPhong`|Specular highlights|
|Custom|Define your own `Lighting<Name>` function|

### 5.3 Surface Output Structures

**SurfaceOutputStandard:**

```hlsl
struct SurfaceOutputStandard
{
    fixed3 Albedo;      // Base color
    fixed3 Normal;      // Tangent-space normal
    half3 Emission;     // Self-illumination
    half Metallic;      // 0=dielectric, 1=metal
    half Smoothness;    // 0=rough, 1=smooth
    half Occlusion;     // Ambient occlusion
    fixed Alpha;        // Transparency
};
```

---

## 6. URP Shader Structure

### 6.1 Basic URP Unlit

```hlsl
Shader "Custom/URPUnlit"
{
    Properties
    {
        _BaseMap ("Texture", 2D) = "white" {}
        _BaseColor ("Color", Color) = (1,1,1,1)
    }
    
    SubShader
    {
        Tags 
        { 
            "RenderType" = "Opaque"
            "RenderPipeline" = "UniversalPipeline"
        }
        
        Pass
        {
            HLSLPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            
            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
            
            CBUFFER_START(UnityPerMaterial)
                float4 _BaseMap_ST;
                half4 _BaseColor;
            CBUFFER_END
            
            TEXTURE2D(_BaseMap);
            SAMPLER(sampler_BaseMap);
            
            struct Attributes
            {
                float4 positionOS : POSITION;
                float2 uv : TEXCOORD0;
            };
            
            struct Varyings
            {
                float4 positionCS : SV_POSITION;
                float2 uv : TEXCOORD0;
            };
            
            Varyings vert(Attributes IN)
            {
                Varyings OUT;
                OUT.positionCS = TransformObjectToHClip(IN.positionOS.xyz);
                OUT.uv = TRANSFORM_TEX(IN.uv, _BaseMap);
                return OUT;
            }
            
            half4 frag(Varyings IN) : SV_Target
            {
                half4 color = SAMPLE_TEXTURE2D(_BaseMap, sampler_BaseMap, IN.uv);
                return color * _BaseColor;
            }
            ENDHLSL
        }
    }
}
```

> [!important] SRP Batcher Use `CBUFFER_START(UnityPerMaterial)` for material properties to enable SRP Batcher compatibility.

---

## 7. Shader Graph

Visual node-based shader editor available in URP and HDRP.

### 7.1 Master Nodes (Targets)

|Target|Description|
|---|---|
|**Unlit**|No lighting calculations|
|**Lit**|PBR lighting|
|**Sprite Lit/Unlit**|2D sprite rendering|
|**Decal**|Decal projection|

### 7.2 Common Node Categories

- **Artistic:** Color adjustments, blending
- **Channel:** Split/combine RGBA
- **Input:** Properties, time, position
- **Math:** Operations, trigonometry
- **Procedural:** Noise, shapes
- **UV:** Manipulation, projection
- **Utility:** Preview, custom function

---

## 8. Render States

### 8.1 Blend Modes

```hlsl
// Syntax: Blend SrcFactor DstFactor
Blend SrcAlpha OneMinusSrcAlpha  // Traditional transparency
Blend One One                      // Additive
Blend DstColor Zero                // Multiplicative
Blend One OneMinusDstColor         // Soft additive
```

**Blend Factors:**

|Factor|Description|
|---|---|
|One|1|
|Zero|0|
|SrcColor|Source color|
|DstColor|Destination color|
|SrcAlpha|Source alpha|
|DstAlpha|Destination alpha|
|OneMinusSrc/DstColor/Alpha|1 - value|

### 8.2 Depth & Stencil

```hlsl
ZWrite On          // Write to depth buffer
ZWrite Off         // Don't write (for transparent)

ZTest LEqual       // Pass if depth <= buffer (default)
ZTest Always       // Always pass
ZTest Less/Greater/etc.

Cull Back          // Cull back faces (default)
Cull Front         // Cull front faces
Cull Off           // No culling (double-sided)
```

---

## 9. Multi-Pass & Keywords

### 9.1 Multiple Passes

```hlsl
SubShader
{
    Pass
    {
        Name "FirstPass"
        // First pass code
    }
    
    Pass
    {
        Name "SecondPass"
        // Second pass code
    }
}
```

### 9.2 Shader Keywords

```hlsl
#pragma multi_compile _ _KEYWORD_A _KEYWORD_B
#pragma shader_feature _OPTIONAL_FEATURE

// In code
#if defined(_KEYWORD_A)
    // Code for keyword A
#endif
```

---

## 10. Performance Tips

|Tip|Description|
|---|---|
|**Minimize passes**|Each pass = additional draw call|
|**Use appropriate precision**|`half` where possible instead of `float`|
|**Reduce texture samples**|Combine textures, use atlases|
|**Avoid branching**|GPUs prefer uniform execution|
|**Use LOD**|Simpler shaders for distant objects|
|**Enable SRP Batcher**|Use proper CBUFFER structure|
|**Profile**|Use Frame Debugger and profiler|

---

## Related Links

- [[Unity Render Texture & Scriptable Render Pipeline Reference]]
- [[Unity Camera Component]]
- [[Unity Post-Processing]]
- [[Unity Lighting]]

---

> [!quote] References
> 
> - Unity Documentation: Writing Shaders
> - Unity Manual: ShaderLab Syntax
> - Catlike Coding: Shader Tutorials
> - Cyanilux: URP Shader Code