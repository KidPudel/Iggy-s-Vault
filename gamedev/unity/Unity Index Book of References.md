# Unity Index Book of References

> A structured learning path for experienced Godot developers transitioning to Unity.


> [!warning] DISCLAIMER
> Before you use this index book, I want you to acknowledge what is the right vs wrong helper document.
> [[Awareness reminder for documents]]

---

# Unity Fundamentals

> [!warning] Pre-struggle for experienced

Core concepts you need before writing any code.

1. [[GameObject and Component Model]] — The fundamental paradigm: composition over inheritance, when to use child GameObjects vs components
2. [[MonoBehaviour Reference]] — What MonoBehaviour is, when Awake/Start/Update run, critical differences from Godot
3. [[Transform and Hierarchy]] — Position/rotation/scale, parent-child relationships, local vs world space
4. [[Prefabs]] — Reusable templates, instantiation, variants (Unity's PackedScene equivalent)
5. [[Serialization and Inspector in Unity]] — How Unity saves data, `[SerializeField]`, what shows in Inspector
6. [[ScriptableObjects]] — Data containers, why never to modify at runtime, event channels and patterns
7. [[Scene Management in Unity]] — Loading scenes, additive loading, why to avoid `DontDestroyOnLoad`
8. [[Reference Objects in Unity]] — Referencing/Accessing other parts to of code (Static, Singleton, Service Locator, Dependency Injection)
9. [[Communication Patterns in Unity]] — Events, UnityEvents, ScriptableObject channels (Unity has no built-in signals)
10. [[Execution Order in Unity]] — Script execution is undefined by default, how to control it
11. [[Input System in Unity]] — Modern Input system in Unity

---

# Architecture Patterns

> [!warning] Post-struggle only

How to structure your project for maintainability.

1. [[Scene Management in Unity]] — Pattern showing scene management and managers persistency (global game objects that are not static ).
2. [[Communication Patterns in Unity]] — Events, UnityEvents, ScriptableObject channels (Unity has no built-in signals)
3. [[Component-Based Design in Unity]] — Single responsibility, reusable components, the 70/30 rule
4. [[Data-Driven Design in Unity]] — Data/State/Representation separation, ScriptableObjects as definitions
5. [[Layered Architecture in Unity]] — Systems/Managers/Components, separating simulation from presentation
6. [[Single Entry Point in Unity]] — Controlled initialization, empty scenes, eliminating race conditions
7. [[Project Structure in Unity]] — Folder organization, feature-based structure, naming conventions

---

# Reference

> [!warning] Pre-struggle for experienced

Quick lookup when you need it.

1. [[Common Patterns Cookbook in Unity]] — Patterns for Singleton, object pool, state machine, save/load, and more
2. [[Unity Component Reference for Godot Developer]] — Rigidbody, Colliders, Renderers, UI, Audio
3. [[Unity Static Classes and Singletons]] — Input, Time, Physics, SceneManager, Mathf, Random. Including Tag, Layer, Physics
4. [[Essential Unity Packages and Assets]] — Input System, TextMeshPro, DOTween, Cinemachine
5. [[Unity Fundamentals for Godot Developers]] — Side-by-side mapping of Godot → Unity concepts
6. [[Unity Gotchas for Godot Developers]] — Important differences between Godot and Unity
7. [[Godot Math to Unity Reference]]

---

# C# reference

> [!warning] Pre-struggle for experienced

1. [[Csharp Quick Reference for Polyglot Programmers]] — C# features with "why" and "when" for polyglot programmers
2. [[Csharp Built-in Interfaces Reference]]
3. [[Game design patterns]]
4. [[cli commands in c sharp]]