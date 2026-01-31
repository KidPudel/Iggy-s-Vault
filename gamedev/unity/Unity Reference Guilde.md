# Unity for Godot Developers — Index

> A structured learning path for experienced Godot developers transitioning to Unity.

---

# Unity Fundamentals

Core concepts you need before writing any code.

1. [[GameObject and Component Model]] — The fundamental paradigm: composition over inheritance, when to use child GameObjects vs components
2. [[MonoBehaviour Reference]] — What MonoBehaviour is, when Awake/Start/Update run, critical differences from Godot
3. [[Transform and Hierarchy]] — Position/rotation/scale, parent-child relationships, local vs world space
4. [[Prefabs]] — Reusable templates, instantiation, variants (Unity's PackedScene equivalent)
5. [[ScriptableObjects]] — Data containers, why never to modify at runtime, event channels and patterns
6. [[Serialization and Inspector in Unity]] — How Unity saves data, `[SerializeField]`, what shows in Inspector
7. [[Scene Management in Unity]] — Loading scenes, additive loading, why to avoid `DontDestroyOnLoad`
8. [[Communication Patterns in Unity]] — Events, UnityEvents, ScriptableObject channels (Unity has no built-in signals)
9. [[Execution Order in Unity]] — Script execution is undefined by default, how to control it
10. [[Unity Gotchas for Godot Developers]] — Specific traps: Start runs once, fake null, SO mutation

---

# Architecture Patterns

How to structure your project for maintainability.

1. [[Component-Based Design in Unity]] — Single responsibility, reusable components, the 70/30 rule
2. [[Layered Architecture in Unity]] — Systems/Managers/Components, separating simulation from presentation
3. [[Data-Driven Design in Unity]] — Data/State/Representation separation, ScriptableObjects as definitions
4. [[Single Entry Point in Unity]] — Controlled initialization, empty scenes, eliminating race conditions
5. [[Project Structure in Unity]] — Folder organization, feature-based structure, naming conventions

---

# Reference

Quick lookup when you need it.

1. [[Unity Fundamentals for Godot Developers]] — Side-by-side mapping of Godot → Unity concepts
2. [[Common Patterns Cookbook in Unity]] — Singleton, object pool, state machine, save/load, and more
3. [[Unity Essential Components Reference]] — Rigidbody, Colliders, Renderers, UI, Audio — when and how
4. [[Unity Static Classes and Singletons]] — Input, Time, Physics, SceneManager, Mathf, Random
5. [[Essential Unity Packages and Assets]] — Input System, TextMeshPro, DOTween, Cinemachine

---

# C# reference

1. [[Csharp Quick Reference for Polyglot Programmers]] — C# features with "why" and "when" for polyglot programmers
2. [[Csharp Built-in Interfaces Reference]]