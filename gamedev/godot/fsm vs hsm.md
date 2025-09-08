### **When FSM is better**

* **Small number of states** (<10).

* **No shared behavior** between states.

* **Transitions are clear and unique**.

* Example:

* A door: Closed ↔ Open ↔ Locked.

* A toggle button: On ↔ Off.

FSM is best when you can draw the whole diagram on a napkin and it’s still readable.

* * *

### **When HSM is better**

* **Many states with shared logic**.

* **Nested structure makes sense** (states belong to categories).

* **You’d otherwise repeat the same transitions in many states**.

* Example:

* Game character:

* Parent Alive → substates Walking, Running, Jumping.

* All share transitions like TakeDamage.

* Parent Dead.

* UI:

* Parent SettingsMenu → substates AudioMenu, VideoMenu, ControlsMenu.

* All share BackToMainMenu.

HSM saves you from copy-pasting transitions/actions across multiple states.

* * *

### **Rule of Thumb**

* Start with **FSM**.

* Switch to **HSM** when you notice:

* Too many states.

* Repeated transitions/actions across states.

* Natural hierarchy in the problem (modes, substates, categories).
