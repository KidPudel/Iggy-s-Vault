## LimboAI in Godot: Fundamental Knowledge Blocks

LimboAI is a powerful Godot addon for creating sophisticated AI using Behavior Trees and State Machines. This reference focuses on the Behavior Tree aspects, which provide a hierarchical and modular way to define AI behavior.

LimboAI is built for **agent AI**: behavior trees + hierarchical state machines to drive autonomous decisions for NPCs or automated systems.
### 1. Behavior Tree (BT)

- **What it is:** A hierarchical, tree-like structure that defines how an AI agent (e.g., an NPC) should make decisions and perform actions. It's a visual, node-based system for crafting AI logic.
    
- **Purpose:** To give "intelligence" to non-player characters (NPCs) in your game. Instead of complex if-else statements in code, you define clear, reusable blocks of behavior.
    
- **Analogy:** Think of it as a flowchart for your AI's brain. The "brain" (root of the tree) constantly evaluates its environment and decides which branch of actions to take.
    

### 2. Agents

- **What it is:** In LimboAI, an "Agent" refers to any character or entity in your game that will be controlled by a Behavior Tree. This is typically your NPCs or enemies.
    
- **Implementation:** Your existing CharacterBody3D (or any Node you want to control) becomes an Agent when you attach a BTPlayer node to it.
    
- **Connection to BT:** The BTPlayer node on your Agent is the bridge between your game object and the Behavior Tree resource. It reads and executes the BT logic.
    

### 3. Blackboard

- **What it is:** A shared memory space accessible by all tasks within a specific Behavior Tree. It's like a whiteboard where different parts of the AI can read and write information.
    
- **Purpose:** To store and retrieve dynamic data that influences the AI's decisions and actions. This prevents tasks from needing to directly access the Agent's code, promoting modularity.
    
- **Example Usage:**
    
    - Storing a target Vector3 position for a MoveToPos task.
        
    - Storing the current "state" of the agent (e.g., "scared", "idle", "attacking") for CheckVar tasks.
        
    - Holding a reference to the player object if the AI detects them.
        

### 4. Tasks

- **What it is:** The fundamental building blocks of a Behavior Tree. Each task represents a single action, condition, or a way to control the flow of other tasks.
    
- **Execution Status:** Every task, when executed, returns one of three statuses:
    
    - **SUCCESS:** The task completed successfully.
        
    - **FAILURE:** The task failed to complete its objective.
        
    - **RUNNING:** The task is currently executing and needs more time to complete.
        
- **Types of Tasks:**
    
    - **Action Tasks (BTAction):** These perform an actual action in the game world (e.g., playing an animation, moving the agent, attacking). You can create custom actions by inheriting from BTAction in GDScript.
        
    - **Condition Tasks:** These check a condition in the game world (e.g., "Is the player close?", "Is my health low?"). They return SUCCESS or FAILURE immediately.
        
    - **Decorator Tasks:** These modify the behavior or return status of their single child task (explained below).
        
    - **Composite Tasks:** These manage the execution flow of their multiple child tasks (explained below).
        

### 5. Composites (Key Flow Control)

These are crucial for structuring your AI's decision-making process.

- **Sequence (->):**
    
    - **Meaning:** A "To-Do List" or an "AND" gate.
        
    - **Behavior:** Executes its child tasks **one by one, in order.**
        
    - **Return:**
        
        - Returns FAILURE as soon as any child task returns FAILURE.
            
        - Returns SUCCESS only if all child tasks return SUCCESS.
            
    - **Analogy:** "First, open the door. Then, walk through. Then, close the door." If any step fails, the whole sequence fails.
        
- **Selector (?):**
    
    - **Meaning:** A "Prioritized Choice" or an "OR" gate.
        
    - **Behavior:** Executes its child tasks **one by one, in order.**
        
    - **Return:**
        
        - Returns SUCCESS as soon as any child task returns SUCCESS.
            
        - Returns FAILURE only if all child tasks return FAILURE.
            
    - **Analogy:** "Can I fly away? (Check state). If yes, fly! If no, can I patrol? If yes, patrol! If no to both, then I have no idea what to do."
        

### 6. Decorators (Task Modifiers)

- **What it is:** Tasks that wrap around a single child task to modify its return status or execution conditions.
    
- **Purpose:** To add flexibility, randomness, or specific constraints to individual actions without altering the core task logic.
    
- **Examples:**
    
    - Probability (50%): If this wraps a "Chill" sequence, the AI will only attempt to chill 50% of the time.
        
    - AlwaysFail: If a task wrapped by this actually succeeds, the decorator will still force it to return FAILURE.
        
    - AlwaysSucceed: If a task wrapped by this fails, the decorator will still force it to return SUCCESS.
        

---

## Simplified LimboAI Workflow:

1. **Design AI Behavior:**
    
    - **High-Level:** What are the main things your AI should do? (e.g., "Fly away if scared," "Patrol if calm"). These are your top-level Selector branches.
        
    - **Sub-Behaviors:** Break down main behaviors into smaller sequences of actions (e.g., "Patrol" -> "Idle," "Wait," "Peck," "Wait," "Move"). These are your Sequences.
        
    - **Conditions:** What triggers each behavior or step? (e.g., "Is player close?" - CheckVar).
        
2. **Create Custom Actions (GDScript):**
    
    - If LimboAI's built-in tasks don't cover your needs, write GDScripts inheriting from BTAction.
        
    - Implement the _tick(delta: float) -> Status function.
        
    - Remember to return SUCCESS, FAILURE, or RUNNING appropriately.
        
3. **Build the Behavior Tree in the Editor:**
    
    - Open your Agent's BTPlayer node in the LimboAI editor.
        
    - Start with a root Selector or Sequence.
        
    - Drag and drop Composites, Actions, and Decorators to construct your tree, matching your design.
        
    - Configure task properties (e.g., animation names for PlayAnimation, variable names for CheckVar/SetVar, probability for Probability).
        
4. **Connect to Blackboard:**
    
    - Define Blackboard variables (e.g., state as "String", pos as "Vector3") in the BTPlayer Inspector or in a BlackboardPlan resource.
        
    - Use SetVar tasks to write values to the Blackboard.
        
    - Use CheckVar tasks to read and evaluate values from the Blackboard.
        
5. **Agent Integration:**
    
    - In your Agent's GDScript (e.g., bird.gd), get references to the BTPlayer (@onready var bt_player = $BTPlayer).
        
    - In the Agent's _physics_process function, add logic to update Blackboard variables based on game events (e.g., if player is close, set bt_player.blackboard.set_var("state", "scared")).
        
    - Call bt_player.restart() when a major state change occurs (e.g., from calm to scared) to force the tree to re-evaluate.
        
6. **Test and Debug:**
    
    - Run your scene.
        
    - Use the LimboAI debugger panel (under "Debugger" tab) to visualize the BT's execution flow in real-time. This is your best friend for understanding and fixing AI bugs.