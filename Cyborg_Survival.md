# Cyborg Survival — Vertical Slice Development Plan

## 1. Project Overview

### Working Concept

**Cyborg Survival** is a sci-fi survival action game where the player's cyborg body functions as equipment, resource infrastructure, and character build.

Players explore a hostile machine world, fight mechanical enemies, salvage technology, replace damaged body parts, install modules, and continuously rebuild their body to survive increasingly dangerous environments.

### Core Fantasy

> **Your body is the survival system.**

Instead of the traditional structure:

```text
Human
├── Weapon
├── Armor
├── Inventory
└── Survival Stats
```

Cyborg Survival is structured around:

```text
                 CYBORG
                    │
        ┌───────────┼───────────┐
        ↓           ↓           ↓
       BODY       MODULES     RESOURCES
        │           │           │
        ↓           ↓           ↓
    Abilities     Build      Energy/Heat
        │           │           │
        └───────────┼───────────┘
                    ↓
                  COMBAT
                    ↓
                  DAMAGE
                    ↓
              BODY DEGRADES
                    ↓
             REPAIR / REPLACE
```

Every major gameplay system should reinforce this fantasy.

---

# 2. Project Goals

## Primary Goal

Develop a **20–30 minute playable vertical slice** that validates the following gameplay hypothesis:

> Body customization creates different builds → builds change combat behavior → combat and exploration provide resources that allow the player to rebuild and improve their body.

## Secondary Goal

Use the project as a technical showcase demonstrating:

- Unreal Engine 5
- C++
- Gameplay Ability System
- Gameplay Tags
- Data-driven gameplay architecture
- Modular character architecture
- Runtime body-part replacement
- Attribute systems
- Gameplay Effects
- Ability granting/removal
- Damage localization
- Equipment systems
- AI
- Save/Load
- Performance profiling
- Technical gameplay architecture

---

# 3. Development Target

## Engine

- Unreal Engine 5
- C++ as primary gameplay implementation
- Blueprint primarily for:
  - Content configuration
  - Animation integration
  - VFX
  - UI
  - Level scripting where appropriate

## Platform

Initial target:

- PC

Potential future targets:

- Console
- Multiplayer

These are **not part of the initial vertical slice**.

---

# 4. Development Timeline

Estimated development time:

**~9 months**

```text
Month 1    Foundation
Month 2    Cyborg Body System
Month 3    Body / Module / Energy / Heat Integration
Month 4    Combat Prototype
Month 5    Loot / Repair / Progression
Month 6    Enemy AI / Encounters
Month 7    Environment / Exploration
Month 8    Vertical Slice Assembly
Month 9    Polish / Optimization
```

Major validation milestone:

> **End of Month 4 — Playable Prototype Gate**

---

# 5. Vertical Slice Scope

The vertical slice should contain approximately:

| Content | Target |
|---|---:|
| Player Character | 1 |
| Environment | 1 |
| Hub | 1 |
| Body Parts | 8–12 |
| Modules | 10–15 |
| Weapons | 2–3 |
| Major Abilities | ~6 |
| Normal Enemy Archetypes | 3 |
| Elite/Boss | 1 |
| Gameplay Duration | 20–30 minutes |

These numbers should be treated as **hard scope limits**, not minimum requirements.

---

# 6. Core Gameplay Loop

```text
                HUB
                 │
            Configure Body
                 │
                 ↓
              Explore
                 │
                 ↓
              Scavenge
                 │
          ┌──────┴──────┐
          ↓             ↓
        Loot          Enemy
          │             │
          │           Combat
          │             │
          └──────┬──────┘
                 ↓
              Damage
                 │
                 ↓
        Body Part Degradation
                 │
          ┌──────┼──────┐
          ↓      ↓      ↓
       Repair Replace Salvage
                 │
                 ↓
             New Build
                 │
                 ↓
          Harder Encounter
                 │
                 ↓
            Better Tech
                 │
                 ↓
               HUB
```

The loop should make the player repeatedly ask:

- Should I keep my current body part?
- Should I replace it?
- Should I repair it?
- Should I salvage it?
- Is the stronger part worth its energy consumption?
- Can my cooling system handle the additional heat?
- Which abilities do I gain or lose?
- Is this build suitable for the next encounter?

---

# 7. Gameplay Pillars

| Pillar | Priority |
|---|---:|
| Cyborg Body System | P0 |
| Energy System | P0 |
| Heat System | P0 |
| Combat | P0 |
| Body Damage / Integrity | P0 |
| Module / Buildcraft | P0 |
| Loot / Salvage | P1 |
| Repair | P1 |
| Progression | P1 |
| Exploration | P1 |
| Survival Systems | P2 |
| Open World | Future |

The **Cyborg Body System** is the central system.

---

# 8. Cyborg Body Structure

The character consists of six primary body parts:

```text
CyborgBody
│
├── Head
├── Torso
├── LeftArm
├── RightArm
├── LeftLeg
└── RightLeg
```

Body parts are not purely cosmetic.

Each body part may influence:

- Attributes
- Energy consumption
- Heat generation
- Armor
- Movement
- Weapon compatibility
- Abilities
- Module capacity
- Damage behavior

---

# 9. Module Slot Structure

Long-term target:

```text
Spine
├── Slot 01
├── Slot 02
└── Slot 03

Shoulders
├── Left
└── Right

Arms
├── Left Slot 01
├── Left Slot 02
├── Left Slot 03
├── Right Slot 01
├── Right Slot 02
└── Right Slot 03

Legs
├── Left Thigh
├── Left Lower Leg
├── Right Thigh
└── Right Lower Leg
```

The vertical slice does **not** need to fully populate every slot.

---

# 10. High-Level Character Architecture

```text
ACyborgCharacter
│
├── UAbilitySystemComponent
├── UCyborgBodyComponent
├── UInventoryComponent
├── UEquipmentComponent
└── UInteractionComponent
```

Responsibilities should remain separated.

`ACyborgCharacter` should coordinate systems rather than contain all gameplay logic.

---

# 11. Data Architecture

A strict separation should exist between:

```text
Definition Data
      ↓
Runtime Instance
      ↓
Presentation
```

Example:

```text
UCyborgBodyPartDefinition
          │
          ↓
FBodyPartInstance
          │
          ↓
BodyPartVisual
```

Gameplay state must not depend directly on visual implementation.

---

# 12. Body Part Definition

Body-part configuration should use `UPrimaryDataAsset`.

Conceptual structure:

```text
UCyborgBodyPartDefinition
    : UPrimaryDataAsset

ID

BodyPartType

Visual
├── SkeletalMesh
├── Material
└── VFX

BaseAttributes

EnergyProfile
HeatProfile

GrantedAbilities[]

GameplayEffects[]

ModuleSlots[]

GameplayTags[]
```

Example:

```text
HydraulicArm_MK2

Type:
LeftArm

Integrity:
120

Armor:
30

Strength:
+25

EnergyConsumption:
+15%

HeatGeneration:
+20%

Abilities:
PowerPunch
HeavyGrab

Slots:
Offensive
Utility
Utility
```

---

# 13. Body Part Runtime State

Runtime state must remain separate from the definition.

```text
FBodyPartInstance

DefinitionID

CurrentIntegrity

DamageState

InstalledModules[]

RuntimeModifiers[]
```

Do not store mutable player state inside Data Assets.

---

# 14. Body Part Lifecycle

Initial implementation:

```text
Healthy
   ↓
Damaged
   ↓
Disabled
```

Future implementation may support:

```text
Healthy
   ↓
Damaged
   ↓
Critical
   ↓
Disabled
   ↓
Destroyed
```

The first prototype should prioritize gameplay consequences over complex damage simulation.

---

# 15. Localized Body Damage

Damage should not simply modify a global HP value.

Target pipeline:

```text
Attack
  ↓
Hit Result
  ↓
Resolve Body Region
  ↓
Damage Calculation
  ↓
BodyPartInstance
  ↓
Integrity -= Damage
  ↓
Damage State Evaluation
  ↓
Gameplay Consequence
```

---

# 16. Body Damage Consequences

## Arm

```text
Left Arm
   ↓
Disabled
```

Possible consequences:

- Remove arm-specific abilities
- Disable certain weapons
- Reduce weapon stability
- Prevent two-handed weapon usage

## Leg

```text
Left Leg
   ↓
Damaged
```

Possible consequences:

- Movement speed reduction
- Increased sprint energy cost
- Reduced dodge performance

## Torso

Possible consequences:

- Reduced maximum energy
- Reduced cooling performance
- Reduced survivability

Body damage should create gameplay decisions rather than functioning purely as another health bar.

---

# 17. Gameplay Ability System

GAS should provide the backbone for:

- Attributes
- Abilities
- Gameplay Effects
- Gameplay Tags
- Status Effects
- Ability restrictions
- Gameplay feedback

---

# 18. Attribute Sets

## Vital Attributes

```text
UVitalAttributeSet

Health
MaxHealth
Armor
```

## Energy Attributes

```text
UEnergyAttributeSet

Energy
MaxEnergy
EnergyRegen
EnergyConsumption
```

## Heat Attributes

```text
UHeatAttributeSet

Heat
MaxHeat
HeatGeneration
HeatDissipation
```

---

# 19. Gameplay Tags

Initial hierarchy:

```text
State
├── Alive
├── Dead
├── Overheated
└── LowEnergy

Body
├── Arm
│   ├── Left
│   └── Right
├── Leg
│   ├── Left
│   └── Right
└── State
    ├── Healthy
    ├── Damaged
    └── Disabled

Ability
├── Combat
│   ├── Melee
│   └── Ranged
├── Mobility
└── Utility

Module
├── Offensive
├── Defensive
├── Utility
└── Mobility

Damage
├── Physical
├── Energy
├── EMP
└── Thermal
```

Avoid creating tags without an actual gameplay requirement.

---

# 20. Energy System

Energy represents the cyborg's **power budget**, not merely stamina.

```text
Energy Generation
      │
      ↓
┌─────────────────┐
│   Energy Pool   │
└────────┬────────┘
         │
 ┌───────┼───────────┐
 ↓       ↓           ↓
Move   Weapons     Modules
 ↓       ↓           ↓
Dash   Abilities   Sensors
```

Example:

```text
Reactor Output       100

Leg Consumption      -15
Arm Consumption      -10
Shield Consumption   -20
Sensor Consumption    -5

Available Energy      50
```

Ability:

```text
Power Punch

Energy:
-25

Heat:
+20
```

A powerful component should therefore carry an operational cost.

---

# 21. Heat System

Heat provides counter-pressure against maximizing raw power.

```text
Powerful Components
        ↓
Higher Energy Consumption
        ↓
Higher Heat Generation
        ↓
Cooling Requirement
        ↓
Module Opportunity Cost
```

### Heavy Build

```text
Damage       ██████████
Armor        █████████
Mobility     ███
Energy Use   █████████
Heat         ██████████
```

### Light Build

```text
Damage       ██████
Armor        ███
Mobility     █████████
Energy Use   ████
Heat         ███
```

A component with larger raw stats should not automatically be superior.

---

# 22. Module System

```text
UCyborgModuleDefinition

ID
ModuleType
CompatibleSlots[]

AttributeModifiers[]
GrantedAbilities[]
GameplayEffects[]
GameplayTags[]

EnergyCost
HeatGeneration
```

Module categories:

### Offensive

- Hydraulic Booster
- Targeting Processor
- Weapon Amplifier

### Defensive

- Armor Plate
- Energy Shield
- Damage Distributor

### Utility

- Cooling Unit
- Battery
- Scanner
- Repair Module

### Mobility

- Jump Booster
- Dash Actuator
- Stabilizer

Prototype target:

**5–8 modules.**

Vertical slice target:

**10–15 modules.**

---

# 23. Module Integration

```text
Install Module
      │
      ├── Modify Attribute
      ├── Grant Ability
      ├── Apply Gameplay Effect
      └── Add Gameplay Tag
```

Example:

```text
Hydraulic Booster

Strength:
+20

Energy Consumption:
+10%

Heat Generation:
+15%

Granted Ability:
PowerPunch
```

---

# 24. Buildcraft

Different configurations must create different gameplay behavior.

## Heavy / Tank Build

```text
Heavy Torso
Hydraulic Arms
Heavy Legs

Armor Module
Cooling Module
```

Characteristics:

```text
Armor        █████████
Damage       ████████
Mobility     ███
Energy Use   ████████
Heat         █████████
```

## Mobility Build

```text
Light Torso
Precision Arm
Servo Legs

Dash Module
Battery Module
```

Characteristics:

```text
Armor        ███
Damage       █████
Mobility     █████████
Energy Use   ██████
Heat         ████
```

Build differences should be perceptible through gameplay, not only numerical stats.

---

# 25. Combat System

Initial combat scope:

```text
Light Attack
Heavy Attack
Ranged Attack
Dodge
Sprint
Body Ability
Module Ability
```

Do not build a large combo system during the prototype phase.

Combat pipeline:

```text
Input
 ↓
Gameplay Ability
 ↓
Animation
 ↓
Hit Detection
 ↓
Gameplay Effect
 ↓
Damage Calculation
 ↓
Body Region
 ↓
BodyPart Integrity
 ↓
Gameplay Consequence
```

Combat must directly interact with the Body System.

---

# 26. Weapons

Vertical slice target:

## Rifle

General-purpose ranged weapon.

## Energy Weapon

Advantages:

- High damage
- Effective against certain armor

Disadvantages:

```text
Energy Consumption ↑↑
Heat Generation ↑↑
```

## Melee Weapon

Lower energy requirement, but performance depends more strongly on:

- Arm type
- Strength
- Body condition

Weapon choice therefore participates in the same build economy as body parts and modules.

---

# 27. Inventory

Avoid complex inventory mechanics during the prototype.

```text
Inventory
│
├── Equipment
├── Body Parts
├── Modules
├── Weapons
└── Resources
```

Item categories:

```text
Resource
BodyPart
Module
Consumable
Weapon
```

A grid/Tarkov-style inventory is not required.

---

# 28. Loot & Salvage

```text
Enemy / Container
       ↓
┌──────┼─────────┐
↓      ↓         ↓
Scrap  Module   Body Part
```

Additional loot may include:

- Energy Cells
- Machine Components
- Rare Technology

Damaged equipment creates a decision:

```text
              Damaged Arm
                   │
       ┌───────────┼───────────┐
       ↓           ↓           ↓
     Equip       Repair      Salvage
                               ↓
                           Materials
```

The player should repeatedly evaluate:

> Is this item more valuable as equipment or as resources?

---

# 29. Repair System

Prototype:

```text
Body Part Damage
      ↓
Repair Station
      ↓
Consume Scrap
      ↓
Restore Integrity
```

Future possibilities:

- Field Repair
- Repair Quality
- Component Replacement
- Permanent Damage
- Maintenance

These are outside the initial vertical slice.

---

# 30. Interaction System

Generic interface:

```cpp
ICyborgInteractable
```

Potential implementations:

```text
Loot
Container
Door
Repair Station
Upgrade Station
Body Part
Terminal
```

Pipeline:

```text
Trace
 ↓
Interactable
 ↓
CanInteract()
 ↓
Interact()
```

Do not force every interaction through GAS.

---

# 31. Exploration

Initial environment:

## Abandoned Industrial Complex

```text
                 HUB
                  │
             Entry Gate
                  │
          ┌───────┴───────┐
          ↓               ↓
      Scrap Yard        Factory
          │               │
     Light Enemy       Heavy Enemy
          │               │
          └───────┬───────┘
                  ↓
              Reactor Area
                  ↓
                Elite
                  ↓
              Rare Tech
```

The level should be semi-linear.

Open-world development is outside the vertical-slice scope.

---

# 32. Enemy Design

Vertical slice:

- 3 normal archetypes
- 1 elite/boss

## Drone

```text
Fast
Ranged
Low HP
```

Potential counters:

- Precision
- Mobility
- EMP

## Worker Machine

```text
Melee
Aggressive
Medium Armor
```

Purpose:

Test close-range combat and body damage.

## Heavy Machine

```text
Slow
High Armor
High Damage
```

Potential counters:

- Energy weapons
- Mobility
- Weak-point targeting

## Elite / Boss

Should combine existing mechanics rather than introduce an entirely new system.

Its purpose is to validate whether accumulated build decisions matter.

---

# 33. AI Architecture

Use Unreal's existing AI infrastructure.

```text
EnemyCharacter
│
├── AbilitySystem
├── AIController
├── Perception
└── CombatComponent
```

Behavior:

```text
AIController
      ↓
StateTree / Behavior Tree
      ↓
Patrol
      ↓
Investigate
      ↓
Engage
      ↓
Attack
      ↓
Reposition
```

Do not build a custom AI framework unless a concrete requirement appears.

---

# 34. Modular Character Rendering

Gameplay Body System and rendering remain independent.

```text
Gameplay

BodyPartInstance
```

versus:

```text
Presentation

BodyPartVisual
```

Initial rendering:

```text
Master Skeleton
│
├── Head Mesh
├── Torso Mesh
├── Left Arm Mesh
├── Right Arm Mesh
├── Left Leg Mesh
└── Right Leg Mesh
```

Use Unreal modular-character workflows.

Do not prioritize runtime skeletal mesh merging during the prototype.

Future optimization may become:

```text
Body Part Configuration
        ↓
Character Rendering System
        ↓
Skeletal Mesh Merge
        ↓
Combined Mesh
```

The gameplay model should remain independent of this optimization.

---

# 35. Save System

Save stable IDs and runtime state rather than treating UObject graphs as the persistence format.

```text
PlayerSave
│
├── EquippedBodyPartIDs
├── BodyPartIntegrity
├── InstalledModuleIDs
├── Inventory
├── Resources
├── UnlockedTechnology
└── ProgressionFlags
```

Example:

```text
LeftArm
{
    Definition = "Arm.Hydraulic.MK2"
    Integrity = 72

    Modules =
    [
        "Module.Cooling.Small"
    ]
}
```

---

# 36. Presentation Layer

```text
Gameplay
   ↓
GameplayCue / Event
   ↓
Presentation
   │
   ├── Animation
   ├── VFX
   ├── SFX
   ├── Camera
   └── Material Feedback
```

Example:

```text
BodyPart.Disabled
       ↓
GameplayCue
       ↓
Sparks
Smoke
Damage Material
Mechanical Failure SFX
```

---

# 37. Development Roadmap

## Month 1 — Foundation

### Week 1 — Project Setup

Tasks:

- Create Unreal Engine C++ project
- Setup Git repository
- Define source/content structure
- Establish coding conventions
- Configure Gameplay Tags
- Setup base Data Asset architecture
- Establish logging/debug conventions

### Week 2 — Character

Implement:

- `ACyborgCharacter`
- Enhanced Input
- Character movement
- Camera
- Basic locomotion

**Deliverable:**

> Playable third-person character.

### Week 3 — GAS Foundation

Implement:

- Ability System Component
- Attribute Sets
- Health
- Energy
- Heat
- Basic Gameplay Effects
- Initial Gameplay Tags

Validate:

```text
Sprint
 ↓
Energy Consumption
```

and:

```text
Ability
 ↓
Heat Generation
 ↓
Overheat
 ↓
Ability Restriction
```

### Week 4 — Interaction / Inventory

Implement:

- Interaction interface
- Interaction trace
- Pickup
- Basic inventory
- Item definition

**Month 1 Milestone:**

```text
Character
 ↓
Explore
 ↓
Interact
 ↓
Pickup
 ↓
Inventory

+

Energy / Heat Foundation
```

---

# 38. Month 2 — Cyborg Body System

## Week 5

Implement:

- BodyPartType
- BodyPartDefinition
- BodyPartInstance
- BodyPartSlot

## Week 6

Implement:

```text
UCyborgBodyComponent

InstallBodyPart()
RemoveBodyPart()
ReplaceBodyPart()
GetBodyPart()
HasBodyPart()
```

## Week 7

Implement modular visuals:

```text
Master Skeleton
      ↓
Runtime Body Part Swap
```

Target:

> Replace LeftArm A with LeftArm B during runtime while animation continues correctly.

## Week 8

Implement:

- ModuleDefinition
- ModuleInstance
- ModuleSlot
- InstallModule
- RemoveModule

**Month 2 Milestone:**

```text
Runtime Body Swap
      ↓
Visual Changes
      +
Gameplay Data Changes
```

---

# 39. Month 3 — System Integration

## Week 9

Connect Body Parts to GAS:

```text
Body Part
   ↓
Attribute Modifier
   ↓
Granted Ability
   ↓
Gameplay Tags
```

## Week 10

Implement:

- Energy consumption
- Heat generation
- Heat dissipation
- Overheat
- Low-energy behavior

## Week 11

Implement:

- Body integrity
- Localized damage
- Healthy/Damaged/Disabled states
- Gameplay consequences

## Week 12

Integrate modules with:

- Attributes
- Energy
- Heat
- Abilities
- Body Parts

**Month 3 Milestone:**

```text
BODY
 ↓
MODULE
 ↓
ENERGY / HEAT
 ↓
ABILITY
 ↓
DAMAGE
 ↓
BODY
```

This must form a closed gameplay loop before proceeding.

---

# 40. Month 4 — Combat Prototype

## Week 13

Implement melee:

- Light Attack
- Heavy Attack
- Hit Detection
- Damage

## Week 14

Implement ranged combat:

- Rifle
- Basic aiming
- Projectile or hitscan
- Damage Effects

## Week 15

Implement localized damage:

```text
Hit
 ↓
Body Region
 ↓
Body Part
 ↓
Integrity
 ↓
Damage State
```

## Week 16

Implement:

- Basic enemy
- Basic AI
- Enemy attacks
- Player damage
- Simple combat encounter

---

# 41. Prototype Gate

At the end of Month 4, temporarily stop feature development.

The prototype must demonstrate:

```text
Small Arena
     │
     ↓
Enemy
     │
     ↓
Combat
     │
     ↓
Body Damage
     │
     ↓
Loot
     │
     ↓
Replace Body Part
     │
     ↓
Build Changes
     │
     └──────────→ Combat
```

Primary validation question:

> Does finding and installing a new body part make the player excited to test how their build changes?

If the answer is **No**, redesign the core systems before adding content.

---

# 42. Month 5 — Loot / Repair / Progression

## Week 17

Implement:

- Loot tables
- Enemy drops
- Container loot

## Week 18

Implement:

- Salvage
- Resource conversion

## Week 19

Implement:

- Repair Station
- Repair costs
- Integrity restoration

## Week 20

Implement:

- Upgrade Station
- Basic progression
- Body configuration UI

**Month 5 Milestone:**

```text
Fight
 ↓
Loot
 ↓
Damage
 ↓
Repair / Replace / Salvage
 ↓
Configure Build
 ↓
Fight
```

---

# 43. Month 6 — AI and Encounter Design

## Weeks 21–22

Implement Drone.

## Week 23

Implement Worker Machine.

## Week 24

Implement Heavy Machine.

## Weeks 25–26

Focus on:

- Encounter composition
- Enemy combinations
- Difficulty
- Build counters
- Combat balancing

Goal:

> Different body configurations should perform differently against different enemy compositions.

---

# 44. Month 7 — Environment / Exploration

Create the Industrial Complex.

Areas:

```text
Hub
 ↓
Entry Gate
 ↓
Scrap Yard / Factory
 ↓
Reactor Area
 ↓
Elite Encounter
```

Add:

- Loot placement
- Containers
- Enemy encounters
- Environmental hazards
- Exploration paths
- Checkpoints

Do not introduce major new gameplay systems during this phase.

---

# 45. Month 8 — Vertical Slice Assembly

Assemble the complete gameplay loop:

```text
START
 ↓
Configure Cyborg
 ↓
Leave Hub
 ↓
Explore
 ↓
Scavenge
 ↓
Combat
 ↓
Body Damage
 ↓
Loot Body Part / Module
 ↓
Repair / Replace / Salvage
 ↓
New Build
 ↓
Harder Encounter
 ↓
Elite
 ↓
Rare Technology
 ↓
Return to Hub
```

Target gameplay duration:

**20–30 minutes.**

---

# 46. Month 9 — Polish and Optimization

No major gameplay features should be introduced.

## Gameplay Polish

- Combat feedback
- Camera feedback
- Hit reactions
- Damage feedback
- Ability readability

## Presentation

- Animation
- VFX
- SFX
- UI/UX
- Lighting
- Materials
- Environment polish

## Technical

- CPU profiling
- GPU profiling
- Memory profiling
- Loading optimization
- Bug fixing
- Stability
- Packaging

## Balance

- Body parts
- Modules
- Energy
- Heat
- Enemy difficulty
- Loot economy

---

# 47. Final Deliverables

```text
Cyborg Survival
│
├── Playable PC Build
├── 20–30 Minute Vertical Slice
├── Gameplay Trailer
├── Technical Demo
├── Architecture Documentation
└── Technical Case Study
```

---

# 48. Portfolio Milestones

Do not wait until Month 9 before presenting the project.

## Month 1

Show:

> Unreal C++ Gameplay Foundation

## Month 2

Show:

> Runtime Modular Cyborg Body System

## Month 3

Show:

> GAS + Body Part + Energy/Heat Integration

## Month 4

Produce the first major technical showcase:

> **Runtime Modular Cyborg System — Unreal Engine C++ / GAS**

Demo sequence:

```text
Normal Arm
 ↓
Equip Hydraulic Arm
 ↓
Visual Changes
 ↓
Attributes Change
 ↓
Ability Granted
 ↓
Energy / Heat Profile Changes
 ↓
Arm Takes Damage
 ↓
Arm Disabled
 ↓
Ability Removed
 ↓
Find Replacement Arm
 ↓
Install
 ↓
Build Changes
```

---

# 49. Explicitly Out of Scope

Do not implement during the initial vertical slice:

- Multiplayer
- Large open world
- Procedural planet generation
- Vehicles
- Base building
- Complex crafting
- Large technology tree
- Complex hunger system
- Thirst
- Sleep
- Disease
- Large faction system
- NPC simulation
- Dynamic economy
- Complex quest framework
- 30+ enemy types
- 100+ modules
- Runtime arbitrary mesh cutting
- Custom cloth solver
- Custom engine-level inventory framework
- Premature skeletal mesh merging

These belong in the post-vertical-slice backlog.

---

# 50. Post-Vertical-Slice Backlog

Potential future systems:

```text
Open World
Advanced Survival
Crafting
Base Building
Weather
Day / Night Cycle
Procedural Generation
Vehicles
Multiplayer
Factions
NPCs
Quest System
Economy
Dynamic Ecosystem
Advanced Damage
Advanced Repair
Advanced Body Customization
```

Each should only be evaluated after the core Cyborg gameplay has been validated.

---

# 51. Priority Order

```text
1. Cyborg Body System
        ↓
2. Modules / Buildcraft
        ↓
3. Energy / Heat
        ↓
4. Combat
        ↓
5. Body Damage
        ↓
6. Loot / Salvage
        ↓
7. Repair
        ↓
8. Progression
        ↓
9. AI
        ↓
10. Exploration
        ↓
11. Survival
        ↓
12. Open World
```

The project should **not** begin with environment production or generic survival mechanics.

---

# 52. Technical Dependency Order

```text
                FOUNDATION
                    │
                    ↓
               BODY SYSTEM
                    │
                    ↓
              MODULE SYSTEM
                    │
                    ↓
             ENERGY / HEAT
                    │
                    ↓
                 COMBAT
                    │
                    ↓
              BODY DAMAGE
                    │
                    ↓
             LOOT / SALVAGE
                    │
                    ↓
               PROGRESSION
                    │
                    ↓
                   AI
                    │
                    ↓
              ENVIRONMENT
                    │
                    ↓
             VERTICAL SLICE
```

Do not invert this dependency chain by producing large amounts of environment/content before validating the Body System.

---

# 53. Suggested Source Structure

```text
Source/
└── CyborgSurvival/
    │
    ├── Core/
    │
    ├── Character/
    │   └── CyborgCharacter
    │
    ├── AbilitySystem/
    │   ├── Attributes/
    │   ├── Abilities/
    │   └── Effects/
    │
    ├── Body/
    │   ├── BodyPart/
    │   ├── BodyPartDefinition/
    │   ├── BodyPartInstance/
    │   └── CyborgBodyComponent/
    │
    ├── Modules/
    │   ├── ModuleDefinition/
    │   ├── ModuleInstance/
    │   └── ModuleComponent/
    │
    ├── Combat/
    │   ├── Damage/
    │   ├── Weapons/
    │   └── Targeting/
    │
    ├── Inventory/
    ├── Interaction/
    ├── AI/
    ├── World/
    ├── Save/
    └── UI/
```

---

# 54. Suggested Content Structure

```text
Content/
└── Cyborg/
    │
    ├── Characters/
    ├── BodyParts/
    ├── Modules/
    ├── Abilities/
    ├── Effects/
    ├── Weapons/
    ├── Enemies/
    ├── World/
    ├── UI/
    ├── VFX/
    └── Audio/
```

---

# 55. Definition of Done — Prototype

The prototype is successful when all of the following are true:

### 1. Body Replacement

Replacing a body part produces an immediately noticeable gameplay change.

### 2. Build Diversity

At least two configurations provide meaningfully different playstyles.

### 3. Energy / Heat Trade-off

The player cannot simply equip every highest-stat component without consequences.

### 4. Localized Damage

Damage to individual body parts creates tactical consequences.

### 5. Loot Motivation

The player wants to continue exploring because new parts/modules can change their build.

---

# 56. Definition of Done — Vertical Slice

A new player should be able to experience:

```text
Explore
 ↓
Fight
 ↓
Take Damage
 ↓
Find Technology
 ↓
Evaluate Equipment
 ↓
Repair / Replace / Salvage
 ↓
Create New Build
 ↓
Experience Changed Combat
 ↓
Defeat Stronger Enemy
 ↓
Return to Hub
```

within approximately **20–30 minutes**, without developer explanation.

---

# 57. Project Validation Questions

At each major milestone, evaluate:

1. Does replacing a body part meaningfully change gameplay?
2. Do different builds feel different rather than only having different numbers?
3. Does Energy create meaningful resource pressure?
4. Does Heat create meaningful build trade-offs?
5. Does localized body damage create interesting decisions?
6. Are modules changing build behavior?
7. Does loot create curiosity and motivation?
8. Is repairing versus replacing a meaningful choice?
9. Do enemies encourage different builds?
10. Does the player understand why their build behaves the way it does?

If several answers are **No**, do not solve the problem by adding more content.

Fix the core systems first.

---

# 58. Current Milestones

Assuming development starts in **October 2026**:

| Deadline | Milestone |
|---|---|
| 31/10/2026 | Unreal Gameplay Foundation |
| 30/11/2026 | Modular Cyborg Body System |
| 31/12/2026 | Body + Module + Energy + Heat |
| 31/01/2027 | Combat Prototype / Prototype Gate |
| 31/03/2027 | Complete Core Gameplay Loop |
| 30/06/2027 | Vertical Slice Feature Complete |
| 31/07/2027 | Polished / Portfolio / Pitch Build |

---

# 59. Immediate 30-Day Goal

The first month should **not** focus on:

- Environment art
- Character art
- Open world
- Crafting
- Complex UI
- Advanced AI

Target:

```text
ACyborgCharacter
        +
Gameplay Ability System
        +
Health
        +
Energy
        +
Heat
        +
Interaction
        +
Basic Inventory
        +
BodyPartDefinition
```

At the end of the first month, the technical foundation should be stable enough to begin the Modular Cyborg Body System.

---

# 60. Most Important Milestone

The most important milestone is **not the final vertical slice**.

It is the first playable prototype.

Target demonstration:

```text
Equip Arm A
    ↓
Character Attributes Change
    ↓
Ability A Becomes Available
    ↓
Enter Combat
    ↓
Arm Takes Localized Damage
    ↓
Arm Becomes Disabled
    ↓
Ability A Becomes Unavailable
    ↓
Enemy Drops Arm B
    ↓
Install Arm B
    ↓
New Attributes
    ↓
New Energy / Heat Profile
    ↓
Ability B Becomes Available
    ↓
Combat Strategy Changes
```

If this sequence is enjoyable with placeholder assets, the project has a strong foundation.

If it is not enjoyable, iterate on this loop before expanding the game.

---

# 61. Guiding Principle

> **Build the Cyborg game first. Build the survival game around it second.**

The project must not become a generic survival game with a cyborg character skin.

The player's:

- Body
- Modules
- Energy
- Heat
- Damage
- Repair
- Replacement
- Build configuration

should remain the central source of gameplay decisions throughout development.
