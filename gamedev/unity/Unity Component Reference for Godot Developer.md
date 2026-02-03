# Godot → Unity Component Reference

> Quick translation guide for Godot developers learning Unity. Links to official Unity docs for implementation details.

---

## Movement & Physics

| Godot Node | Unity Component | Docs |
|------------|-----------------|------|
| **CharacterBody3D** | CharacterController | [docs.unity3d.com](https://docs.unity3d.com/ScriptReference/CharacterController.html) |
| **CharacterBody2D** | CharacterController (2D movement via script) | [docs.unity3d.com](https://docs.unity3d.com/ScriptReference/CharacterController.html) |
| **RigidBody3D** | Rigidbody | [docs.unity3d.com](https://docs.unity3d.com/ScriptReference/Rigidbody.html) |
| **RigidBody2D** | Rigidbody2D | [docs.unity3d.com](https://docs.unity3d.com/ScriptReference/Rigidbody2D.html) |

---

## Collision

| Godot Node | Unity Component | Docs |
|------------|-----------------|------|
| **CollisionShape3D (Box)** | BoxCollider | [docs.unity3d.com](https://docs.unity3d.com/ScriptReference/BoxCollider.html) |
| **CollisionShape2D (Box)** | BoxCollider2D | [docs.unity3d.com](https://docs.unity3d.com/ScriptReference/BoxCollider2D.html) |
| **CollisionShape3D (Sphere)** | SphereCollider | [docs.unity3d.com](https://docs.unity3d.com/ScriptReference/SphereCollider.html) |
| **CollisionShape2D (Circle)** | CircleCollider2D | [docs.unity3d.com](https://docs.unity3d.com/ScriptReference/CircleCollider2D.html) |
| **CollisionShape3D (Capsule)** | CapsuleCollider | [docs.unity3d.com](https://docs.unity3d.com/ScriptReference/CapsuleCollider.html) |
| **CollisionShape2D (Capsule)** | CapsuleCollider2D | [docs.unity3d.com](https://docs.unity3d.com/ScriptReference/CapsuleCollider2D.html) |
| **CollisionShape3D (Mesh)** | MeshCollider | [docs.unity3d.com](https://docs.unity3d.com/ScriptReference/MeshCollider.html) |
| **CollisionPolygon2D** | PolygonCollider2D | [docs.unity3d.com](https://docs.unity3d.com/ScriptReference/PolygonCollider2D.html) |
| **Area3D** | Collider with `isTrigger = true` | [docs.unity3d.com](https://docs.unity3d.com/ScriptReference/Collider-isTrigger.html) |
| **Area2D** | Collider2D with `isTrigger = true` | [docs.unity3d.com](https://docs.unity3d.com/ScriptReference/Collider2D-isTrigger.html) |

---

## Rendering 3D

| Godot Node | Unity Component | Docs |
|------------|-----------------|------|
| **MeshInstance3D** | MeshFilter + MeshRenderer | [MeshFilter](https://docs.unity3d.com/ScriptReference/MeshFilter.html) / [MeshRenderer](https://docs.unity3d.com/ScriptReference/MeshRenderer.html) |
| **Skeleton3D (skinned mesh)** | SkinnedMeshRenderer | [docs.unity3d.com](https://docs.unity3d.com/ScriptReference/SkinnedMeshRenderer.html) |
| **Line3D** | LineRenderer | [docs.unity3d.com](https://docs.unity3d.com/ScriptReference/LineRenderer.html) |
| **Trail3D** | TrailRenderer | [docs.unity3d.com](https://docs.unity3d.com/ScriptReference/TrailRenderer.html) |

---

## Rendering 2D

| Godot Node | Unity Component | Docs |
|------------|-----------------|------|
| **Sprite2D** | SpriteRenderer | [docs.unity3d.com](https://docs.unity3d.com/ScriptReference/SpriteRenderer.html) |
| **TileMap** | Tilemap + TilemapRenderer | [docs.unity3d.com](https://docs.unity3d.com/Manual/class-Tilemap.html) |

---

## Animation

| Godot Node | Unity Component | Docs |
|------------|-----------------|------|
| **AnimationPlayer** | Animator | [docs.unity3d.com](https://docs.unity3d.com/ScriptReference/Animator.html) |
| **AnimationTree** | Animator (with blend trees) | [docs.unity3d.com](https://docs.unity3d.com/Manual/class-BlendTree.html) |

---

## Audio

| Godot Node | Unity Component | Docs |
|------------|-----------------|------|
| **AudioStreamPlayer** | AudioSource | [docs.unity3d.com](https://docs.unity3d.com/ScriptReference/AudioSource.html) |
| **AudioStreamPlayer3D** | AudioSource (with `spatialBlend = 1`) | [docs.unity3d.com](https://docs.unity3d.com/ScriptReference/AudioSource-spatialBlend.html) |
| **AudioListener** | AudioListener | [docs.unity3d.com](https://docs.unity3d.com/ScriptReference/AudioListener.html) |

---

## UI (Canvas-based in Unity)

| Godot Node | Unity Component | Docs |
|------------|-----------------|------|
| **Control (root)** | Canvas | [docs.unity3d.com](https://docs.unity3d.com/Packages/com.unity.ugui@1.0/manual/UICanvas.html) |
| **TextureRect** | Image | [docs.unity3d.com](https://docs.unity3d.com/Packages/com.unity.ugui@1.0/manual/script-Image.html) |
| **Label** | TextMeshPro - Text (UI) | [docs.unity3d.com](https://docs.unity3d.com/Packages/com.unity.textmeshpro@3.0/manual/index.html) |
| **Button** | Button | [docs.unity3d.com](https://docs.unity3d.com/Packages/com.unity.ugui@1.0/manual/script-Button.html) |
| **Slider** | Slider | [docs.unity3d.com](https://docs.unity3d.com/Packages/com.unity.ugui@1.0/manual/script-Slider.html) |
| **CheckBox** | Toggle | [docs.unity3d.com](https://docs.unity3d.com/Packages/com.unity.ugui@1.0/manual/script-Toggle.html) |
| **OptionButton** | Dropdown | [docs.unity3d.com](https://docs.unity3d.com/Packages/com.unity.ugui@1.0/manual/script-Dropdown.html) |
| **LineEdit** | InputField (TMP) | [docs.unity3d.com](https://docs.unity3d.com/Packages/com.unity.ugui@1.0/manual/script-InputField.html) |
| **ScrollContainer** | ScrollRect | [docs.unity3d.com](https://docs.unity3d.com/Packages/com.unity.ugui@1.0/manual/script-ScrollRect.html) |

---

## Camera & Lighting

| Godot Node | Unity Component | Docs |
|------------|-----------------|------|
| **Camera3D** | Camera | [docs.unity3d.com](https://docs.unity3d.com/ScriptReference/Camera.html) |
| **Camera2D** | Camera (with Orthographic projection) | [docs.unity3d.com](https://docs.unity3d.com/ScriptReference/Camera-orthographic.html) |
| **DirectionalLight3D** | Light (Type: Directional) | [docs.unity3d.com](https://docs.unity3d.com/Manual/Lighting.html) |
| **PointLight3D** | Light (Type: Point) | [docs.unity3d.com](https://docs.unity3d.com/Manual/Lighting.html) |
| **SpotLight3D** | Light (Type: Spot) | [docs.unity3d.com](https://docs.unity3d.com/Manual/Lighting.html) |

---

## Navigation (AI Pathfinding)

| Godot Node | Unity Component | Docs |
|------------|-----------------|------|
| **NavigationAgent3D** | NavMeshAgent | [docs.unity3d.com](https://docs.unity3d.com/ScriptReference/AI.NavMeshAgent.html) |
| **NavigationObstacle3D** | NavMeshObstacle | [docs.unity3d.com](https://docs.unity3d.com/ScriptReference/AI.NavMeshObstacle.html) |

---

## Effects

| Godot Node | Unity Component | Docs |
|------------|-----------------|------|
| **GPUParticles3D / CPUParticles3D** | ParticleSystem | [docs.unity3d.com](https://docs.unity3d.com/ScriptReference/ParticleSystem.html) |

---

## Key Differences to Remember

- **Godot**: Node-based inheritance + child composition
- **Unity**: Pure component composition on GameObjects
- **Godot scenes** = **Unity prefabs** (reusable compositions)
- **Unity requires separate MeshFilter + MeshRenderer** (Godot combines in MeshInstance3D)
- **Unity UI is Canvas-based** (different paradigm from Godot's Control nodes)
- **Unity uses isTrigger flag** instead of separate Area nodes

