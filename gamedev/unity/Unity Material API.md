# Unity Material API

Namespace: `UnityEngine`

All `Set*` methods create a material instance if used on `renderer.material`. Use `renderer.sharedMaterial` to modify the asset directly.

## Property Getters/Setters

`GetColor`/`SetColor`, `GetFloat`/`SetFloat`, `GetInt`/`SetInt`, `GetInteger`/`SetInteger`, `GetMatrix`/`SetMatrix`, `GetTexture`/`SetTexture`, `GetTextureOffset`/`SetTextureOffset`, `GetTextureScale`/`SetTextureScale`, `GetVector`/`SetVector`

All accept `(string name)` or `(int nameID)`.

Array variants: `GetColorArray`/`SetColorArray`, `GetFloatArray`/`SetFloatArray`, `GetMatrixArray`/`SetMatrixArray`, `GetVectorArray`/`SetVectorArray`.

## Other Methods

| Method | Description |
|---|---|
| `HasProperty(string name)` | Check if property exists |
| `EnableKeyword(string)` | Enable shader keyword |
| `DisableKeyword(string)` | Disable shader keyword |
| `IsKeywordEnabled(string)` | Check keyword state |
| `CopyPropertiesFromMaterial(Material)` | Copy all properties |
| `renderQueue` (property) | Render queue value |
| `shader` (property) | The shader used |
| `mainTexture` (property) | Shorthand for `_MainTex` |
| `color` (property) | Shorthand for `_Color` |

## Property Name IDs

```csharp
private static readonly int ColorID = Shader.PropertyToID("_Color");
mat.SetColor(ColorID, Color.red);
```

## Common Property Names (URP Lit)

| Name | Type | Description |
|---|---|---|
| `_BaseColor` | Color | Albedo color |
| `_BaseMap` | Texture | Albedo texture |
| `_BumpMap` | Texture | Normal map |
| `_BumpScale` | Float | Normal map scale |
| `_Metallic` | Float | Metallic (0-1) |
| `_Smoothness` | Float | Smoothness (0-1) |
| `_EmissionColor` | Color (HDR) | Emission color |
| `_EmissionMap` | Texture | Emission texture |
| `_Cutoff` | Float | Alpha cutoff |
| `_Surface` | Float | 0 = Opaque, 1 = Transparent |

> [!info]
> Built-in pipeline uses `_Color` and `_MainTex`. URP uses `_BaseColor` and `_BaseMap`.

## MaterialPropertyBlock

Per-renderer overrides without creating material instances.

```csharp
MaterialPropertyBlock mpb = new MaterialPropertyBlock();
Renderer renderer = GetComponent<Renderer>();
renderer.GetPropertyBlock(mpb);
mpb.SetColor("_BaseColor", Color.red);
renderer.SetPropertyBlock(mpb);
renderer.SetPropertyBlock(null);  // clear
```

Same `Get*/Set*` methods as Material, plus `Clear()` and `isEmpty`.

## Sources
- [Material](https://docs.unity3d.com/ScriptReference/Material.html)
- [MaterialPropertyBlock](https://docs.unity3d.com/ScriptReference/MaterialPropertyBlock.html)
