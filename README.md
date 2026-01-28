# Real-Time Resource Management System

![Godot](https://img.shields.io/badge/Engine-Godot_4.x-478cbf?logo=godotengine)
![Language](https://img.shields.io/badge/Language-GDScript_(Python--like)-blue)
![Architecture](https://img.shields.io/badge/Pattern-State_Machine-orange)
![Status](https://img.shields.io/badge/Roadmap-Migrating_to_.NET%2FC%23-purple)

> **Architectural Prototype:** A modular, data-driven simulation designed to model resource allocation (Health/Mana), priority dispatching, and event handling in a real-time environment.
>
> **Note:** Currently implemented in GDScript. The architecture is intentionally designed for a planned migration to **C# / .NET** to leverage strict typing, Interfaces, and LINQ.

## Project Abstract

This project simulates a "Central Dispatcher" (The Player) tasked with managing the integrity of multiple independent Nodes (Agents) under varying stress loads.

The primary engineering goal was to decouple **Logic** from **Data**. Abilities, Stress Events, and Unit Configurations are defined as abstract resources. This allows the simulation engine to process arbitrary effects without requiring changes to the source code—a pattern essential for scalable software.

---

## Core Architecture & Patterns

### Finite State Machine (FSM)
To manage the complex behavior of autonomous units (e.g., Idle $\rightarrow$ Active $\rightarrow$ Cooldown), a reusable State Machine pattern was implemented. This avoids "Spaghetti Code" in the main update loop.

**Code Highlight: Dynamic State Injection**<br/>

The State Machine uses a reflection-like approach to automatically discover and initialize its state nodes at runtime. This allows for loose coupling between the controller and its states.

```gdscript
class_name StateMachine
extends Node

# Dependency Injection via 'init'
func init(entity: Node):
    owner_entity = entity
 
    # Automatically find State nodes attached to this object
    for child in get_children():
        if child is State:
            # Register states in a Dictionary for O(1) access
            states[child.name.to_lower()] = child
            child.Transitioned.connect(on_child_transition)
            if child.has_method("init"):
                child.init(entity)

```

### Data-Driven Design (Strategy Pattern)

Instead of hard-coding logic (e.g., `func cast_fireball()`), the system uses a generic `EffectData` class. This class acts as a configuration container. The engine reads this data to execute the logic, effectively implementing the Strategy Pattern via data.

```gdscript
class_name EffectData
extends Resource

# Enums define strict types for the logic engine
@export var effect_type: EffectEnums.EffectType = EffectEnums.EffectType.HEAL
@export var trigger: EffectEnums.TriggerType = EffectEnums.TriggerType.ON_CAST
@export var target: EffectEnums.TargetType = EffectEnums.TargetType.TARGET

# Configurable logic parameters
@export var is_aoe: bool = false
@export var value: int = 0
@export var interval: float = 0.0 
@export var proc_chance: float = 0.0 

```

---

## Algorithms & Data Handling

### Dynamic Layout Mapping

To visualize the status of monitored nodes, the system must dynamically map data entities to specific UI slots based on their role configuration.

**Code Highlight: Dictionary-based Slot Assignment**<br/>

The system uses a mapping algorithm to assign entities to grid slots. This separates the logical role of a unit (e.g., "Tank" priority) from its visual representation.

```gdscript
# Inside LayoutManager.gd

func setup_layout():
    # Define mapping rules: Which Role goes to which Grid Index?
    var role_map := {
        MemberClassData.RoleType.TANK: [1],      # Priority Slot
        MemberClassData.RoleType.OFFTANK: [3],
        MemberClassData.RoleType.MELEE: [5, 7, 9],
        MemberClassData.RoleType.RANGED: [0, 2, 4, 6, 8]
    }

    # Algorithm: Assign members to slots based on available indices
    var frames_by_index = []
    frames_by_index.resize(10) 

    for member in active_roster:
        # Fetch allowed slots for this member's role
        var indices = role_map.get(member.current_role, [])
        for idx in indices:
            if frames_by_index[idx] == null:
                frames_by_index[idx] = create_container(member)
                break

```

### Priority Filtering (Triage Logic)

The system constantly evaluates the state of the group to make automated decisions. Efficient filtering methods are used to query the dataset without unnecessary iterations.

```gdscript
func get_critical_node() -> MemberClassData:
    var critical_nodes = []
    for member in active_roster:
        # Check integrity threshold (Health < Max)
        if member.current_health < member.max_health:
            critical_nodes.append(member)
            
    if critical_nodes.size() == 0:
        return null
    
    # Return a random node from the critical subset
    return critical_nodes[randi() % critical_nodes.size()]

```

---

## Future Roadmap: .NET Migration

The project serves as a prototype for a high-performance C# implementation.
Key refactoring goals include:

1. **Interfaces (`IState`, `IEffect`):** To enforce stricter contracts than GDScript's duck-typing.
2. **LINQ:** To replace manual loops (like in `get_critical_node`) with expressive queries:
* *Example:* `roster.Where(m => m.Health < m.MaxHealth).OrderBy(m => m.Health).First();`


3. **Events:** Replacing Godot Signals with standard C# Events / Delegates.

---

## Technical Skills Demonstrated

* **System Architecture:** Designing decoupled systems using FSM and Signals.
* **Algorithm Design:** Implementing mapping and filtering logic for dynamic data sets.
* **Data Modeling:** Structuring complex simulation data into reusable Resources.
* **Tooling:** Using the Editor as a CMS for defining simulation logic.
