Tags and attributes are lightweight state markers. They let you drive logic without stuffing everything into giant scripts or deep inheritance chains.

How they’re used in practice:

Tags (CollectionService) are binary labels. They answer the question: “Is this thing part of a category or behavior group?”  
Examples: “Damageable”, “Interactable”, “Enemy”, “Pickup”, “OnFire”.  
A system script listens for objects with a certain tag and applies logic. Think of tags as a switchboard for routing objects into systems.

Attributes are small pieces of per-instance data. They answer: “What parameters or current state does this instance have?”  
Examples: Health = 40, IsStunned = true, SpeedMultiplier = 1.2.  
They’re cheap, replicating, and attach to individual objects without forcing custom modules or cluttering your hierarchy.

Why Roblox devs lean on them:

They cut coupling. Systems don’t reference specific objects; they just react to state.  
They make dynamic behavior trivial: add a tag → system wakes up; remove it → system shuts off.  
They keep your architecture data-driven instead of branching all your code with special cases.