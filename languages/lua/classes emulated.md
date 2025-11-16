Classes are used to define blueprints for creating objects (instances) that share common properties and behaviors. They enable object-oriented programming principles like encapsulation, inheritance, and polymorphism.
**Implementation:** 
Classes in Lua are emulated using [[table]] and [[metatable]]. A "class" table often contains methods and a `__index` metamethod that points back to itself (or a parent class for inheritance). Instances of the class are then created as new tables with their own data, and their [[metatable]] is set to the "class" table to inherit methods.

- **Characteristics:**
    - **Multiple instances:** You can create multiple independent instances (objects) from a single "class" definition, each with its own state.
    - **Stateful:** Instances can hold unique data (fields) that differentiate them from other instances of the same "class."
    - **Inheritance (Emulated):** Metatables allow for the emulation of inheritance, where instances can inherit methods and default values from a parent "class."