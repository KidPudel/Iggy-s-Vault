# Unity URP vs Built-in Pipeline

| Feature | Built-in | URP |
|---|---|---|
| Default shader | Standard | Universal Render Pipeline/Lit |
| Albedo color | `_Color` | `_BaseColor` |
| Albedo texture | `_MainTex` | `_BaseMap` |
| Smoothness | `_Glossiness` | `_Smoothness` |
| Shader include | `UnityCG.cginc` | `Packages/.../Core.hlsl` |
| Vertex struct | `appdata_full v` | `Attributes input` |
| Fragment sig | `fixed4 frag(v2f i)` | `half4 frag(Varyings input)` |
| Object to clip | `UnityObjectToClipPos(v.vertex)` | `TransformObjectToHClip(input.positionOS)` |
| World position | `mul(unity_ObjectToWorld, v.vertex)` | `TransformObjectToWorld(input.positionOS)` |
| World normal | `UnityObjectToWorldNormal(v.normal)` | `TransformObjectToWorldNormal(input.normalOS)` |
| Fog | `UNITY_FOG_COORDS`, `UNITY_APPLY_FOG` | `MixFog(color, fogFactor)` |
| Light access | `_WorldSpaceLightPos0` | `GetMainLight()` |
| Shadow | `SHADOW_ATTENUATION` | `GetMainLight(shadowCoord).shadowAttenuation` |
| SH lighting | `ShadeSH9(normal)` | `SampleSH(normalWS)` |
| Forward pass | `"LightMode" = "ForwardBase"` | `"LightMode" = "UniversalForward"` |
| Batching | Dynamic, GPU instancing | SRP Batcher (default) |

## URP Includes

```hlsl
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Shadows.hlsl"
```

## URP Light Struct

```hlsl
Light mainLight = GetMainLight();
// .direction (half3), .color (half3), .distanceAttenuation (half), .shadowAttenuation (half)

Light mainLight = GetMainLight(shadowCoord);  // with shadows

int count = GetAdditionalLightsCount();
for (int i = 0; i < count; i++)
{
    Light light = GetAdditionalLight(i, positionWS);
}
```

## Sources
- [URP documentation](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@14.0/manual/index.html)
