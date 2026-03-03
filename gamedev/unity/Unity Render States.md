# Unity Render States

## Render Queue Values

| Name | Value | Use |
|---|---|---|
| Background | 1000 | Skybox |
| Geometry | 2000 | Opaque objects (default) |
| AlphaTest | 2450 | Alpha-tested geometry |
| Transparent | 3000 | Transparent (back-to-front) |
| Overlay | 4000 | Lens flares, UI overlays |

## Blend Modes (ShaderLab)

```
Blend SrcAlpha OneMinusSrcAlpha    // Traditional alpha
Blend One One                       // Additive
Blend One OneMinusSrcAlpha          // Premultiplied alpha
Blend DstColor Zero                 // Multiply
Blend Off                           // Opaque
```

## Blend Factors

`One`, `Zero`, `SrcColor`, `SrcAlpha`, `DstColor`, `DstAlpha`, `OneMinusSrcColor`, `OneMinusSrcAlpha`, `OneMinusDstColor`, `OneMinusDstAlpha`

## ZTest / ZWrite / Cull

```
ZWrite On|Off
ZTest LEqual|Less|Greater|GEqual|Equal|NotEqual|Always
Cull Back|Front|Off
```

## Stencil

```
Stencil
{
    Ref [_StencilRef]
    Comp [_StencilComp]
    Pass [_StencilPass]
    Fail [_StencilFail]
    ZFail [_StencilZFail]
    ReadMask [_StencilReadMask]
    WriteMask [_StencilWriteMask]
}
```

Comparison: `Always`, `Never`, `Less`, `LEqual`, `Greater`, `GEqual`, `Equal`, `NotEqual`

Operations: `Keep`, `Zero`, `Replace`, `IncrSat`, `DecrSat`, `Invert`, `IncrWrap`, `DecrWrap`

## Sources
- [Unity Manual: ShaderLab](https://docs.unity3d.com/Manual/SL-Reference.html)
