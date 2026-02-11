# Unity Index Book of References

> [!warning] DISCLAIMER
> [[Awareness reminder for documents]]

---

# Unity Fundamentals

Core concepts and APIs you look up while working.

1. [[GameObject and Component Model]] — Composition paradigm, when to use child GameObjects vs components
2. [[MonoBehaviour Reference]] — Lifecycle methods, coroutines, scripting foundation
3. [[Transform and Hierarchy]] — Position/rotation/scale, parent-child, local vs world space
4. [[Prefabs]] — Reusable templates, instantiation, variants
5. [[Serialization and Inspector in Unity]] — How Unity saves data, `[SerializeField]`, Inspector attributes
6. [[ScriptableObjects]] — Data containers as project assets, Catalog and Identity patterns
7. [[Scene Management in Unity]] — Scene loading API, additive loading *(architecture sections: Socratic)*
8. [[Reference Objects in Unity]] — Singleton, Service Locator, DI implementations *(choosing between them: Socratic)*
9. [[Communication Patterns in Unity]] — C# events, UnityEvents, SO channels syntax *(choosing patterns: Socratic)*
10. [[Execution Order in Unity]] — Script execution order, DefaultExecutionOrder attribute
11. [[Input System in Unity]] — New Input System setup, Input Actions, rebinding
12. [[Single Entry Point in Unity]] — Boot scene technique for controlled initialization, eliminating race conditions

---

# Architecture & Design Thinking

These are thinking patterns. They live in your brain, not in documents. Work through them with your [[Socratic Partner Prompt]].

- Composition and component design (single responsibility, reusable vs entity-specific, the 70/30 split)
- Data-driven architecture (data assets, runtime state, representation, when to separate what)
- What roles can code play and when does it make sense. Code organization (systems, managers, components as roles)
	-  The individual pieces are well-known patterns:
	  - Components — that's just Unity's component model
	  - Systems — that's the Service pattern (centralized state owners), also from ECS thinking
	  - Managers — that's the Mediator pattern (coordinating between unrelated things)
- Project structure and folder organization
- Interface vs abstract class (when to use which in C#)

---

# Reference

Quick lookup.

1. [[Common Patterns Cookbook in Unity]] — Singleton, Object Pool, State Machine, Health, Interaction, Save/Load implementations
2. [[Unity Component Reference for Godot Developer]] — Godot node → Unity component mapping
3. [[Unity Static Classes and Singletons]] — Input, Time, Physics, SceneManager, Mathf, Random
4. [[Essential Unity Packages and Assets]] — Input System, TextMeshPro, DOTween, Cinemachine
5. [[Unity Fundamentals for Godot Developers]] — Side-by-side concept mapping
6. [[Unity Gotchas for Godot Developers]] — Engine-specific traps and differences
7. [[Godot Math to Unity Reference]] — Math API mapping
8. [[Unity Camera Component]] — Camera properties, projection, multi-camera setups
9. [[Unity Post-Processing]] — Volume system, common effects, performance
	1. [[Unity Shaders]] — Shader Graph vs hand-written, material properties
	2. [[Unity Lighting]] — Light types, baking, global illumination, shadows

---

# C# Reference

1. [[Csharp Quick Reference for Polyglot Programmers]] — Language features with "why" and "when"
2. [[Csharp Built-in Interfaces Reference]] — IEnumerable, IDisposable, IEquatable, etc.
3. [[Game design patterns]] — Pattern implementations reference
4. [[cli commands in c sharp]] — dotnet CLI commands
