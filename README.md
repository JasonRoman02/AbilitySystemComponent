## AbilityComponentSystem

## System Overview
This document outlines the implementation of a comprehensive ability management system in Unreal Engine 5.5 using Blueprint Visual Scripting. The system manages activatable abilities with cooldowns, resource costs, buff/debuff mechanics, and projectile-based skills, all integrated with a responsive HUD interface. There are many cases ready for logic and I am willing to go in and fill them out, but I only focused on self-casted and projectile abilities for this assessment.

## Architecture Components
# 1. AC_Abilities (Actor Component)
Purpose: Central hub for all ability-related functionality, managing skill activation, cooldowns, resource consumption, and effect processing.
Key Features:
Data table-driven ability system for easy designer iteration
Multi-target type support (Self, Single Target, Projectile Path, etc.)
Resource management integration (Health/Mana consumption)
Buff/debuff application and removal
Event-driven UI communication
Flexible effect system supporting damage, healing, and projectile spawning
Core Variables:
PlayerAbilitySlots (Array of S_AbilityData): Currently equipped abilities
ActiveCooldowns (Map: String → Float): Tracks remaining cooldown times
ActiveBuffs (Array of S_BuffData): Currently applied temporary effects
ResourceComponent (AC_ResourceComponent Reference): Handles health/mana

# Key Functions:
RegisterAbility(): Loads abilities from data table into player slots
ActivateAbility(): Main ability activation with validation and resource checks
ProcessAbilityEffect(): Handles different effect types (damage, healing, projectile spawning)
ApplyBuffsFromTemplate(): Applies temporary stat modifications
ApplyStatModifications() / RemoveStatModifications(): Applies buffs or Cleans up expired buffs
UpdateAllBuffs(): Tick-based buff management system
UpdateAllCooldowns(): Tick-based cooldown management system

# 2. AC_ResourceComponent (Actor Component)
Purpose: This was not asked for by the assessment, but I felt that I needed a proper system to manage Health, Mana, and Stamina across the board. This is why I made another component. Manages player resources (Health, Mana) with regeneration capabilities and buff interactions.
# Key Features:
Health, Mana and Stamina tracking with configurable maximums
Regeneration system with rate modifications, with flags to determine if regeneration is even possible.
Buff-influenced regeneration (can be disabled/modified by abilities)
Event dispatching for UI updates
Integration with damage and healing systems
# Core Variables:
CurrentHealth/MaxHealth (Float): Player health values
CurrentMana/MaxMana (Float): Player mana values
CurrenStamina/MaxStamina (Float): Player stamina values
HealthRegenRate/ManaRegenRate/StaminaRegenRate (Float): Base regeneration per second
StaminaRegenDelay (Float): Stamina Regeneration could be delayed 
bCanRegenHealth/bCanRegenMana/bCanRegenStamina (Boolean): Regeneration toggle flags

# 3. Data Structure System
S_AbilityData Struct: Complete ability definition including:
Basic info (Name, ID, Description, Icon)
Resource costs and cooldowns
Targeting system (Self, Single Target, Projectile Path)
Range and line-of-sight requirements
Effect arrays for complex ability behaviors
Projectile class specification for ranged abilities
S_AbilityEffect Struct: Ability effect configuring including:
EffectID
Effect Typing and Timing (Spawning Projectile, Instant)
Duration
Tick intervals
Buff arrays for complex buff behaviors
Spawn Actor entry to store what kind of actor will spawn
S_BuffData Struct: Temporary effect system including:
Stat modifications (health regen, movement speed, etc.)
Duration and tick-based effects
Stacking behavior and cleanup mechanisms

# 4. Enhanced Input Integration
# Input Actions:
IA_Ability1, IA_Ability2, IA_Ability3: Keyboard bindings (1, 2, 3 keys)
Routed through player controller to ActivateAbilitySlot() function
Target Selection System:
PROJECTILE_PATH: Uses camera-based line tracing for skill shots
SINGLE_TARGET: Direct actor targeting with validation
SELF: Automatic self-targeting for buffs and healing
There are others, but logic is incomplete, these did fill the requirements
Implemented Abilities
Heal Self (heal_self)
Type: SELF targeting
Resource Cost: 30 Mana
Cooldown: 5 seconds
Effect: 2 HP per second for 5 seconds
Implementation: Direct health regen rate modification through AC_ResourceComponent
Speed Boost (speed_boost)
Type: SELF targeting buff
Resource Cost: 40 Mana
Cooldown: 12 seconds
Duration: 12 seconds
Effect: 30% movement speed increase
Implementation: Temporary buff applied through buff template system interacting with the Character Movement component provided by the ThirdPersonCharacter
Fire Bolt (fire_bolt)
Type: PROJECTILE_PATH skill shot
Resource Cost: 25 Mana
Cooldown: 5 seconds
Range: 1000 units
Effect: 50 damage projectile
Implementation: Spawns BP_FireballProjectile with collision-based damage
Projectile System Architecture
BP_BaseProjectile (Parent Class)
Purpose: Reusable projectile foundation for all ranged abilities. This helped with spawning projectiles and keeping it expandable for all sorts of projectiles.
Components:
Sphere Collision Component (Root): Handles collision detection
Static Mesh Component: Visual representation
Projectile Movement Component: Physics-based movement
# Key Variables:
DamageAmount (Float): Damage to apply on hit
Caster (Actor Reference): Ability user for self-collision avoidance
TargetFiltering (Enum): Determines valid targets (Enemies, Allies, All)
EffectTiming (Enum): Determines how effects land (Instant, On Hit, Over Time)
MaxRange (Float): Auto-destruction distance
BP_FireballProjectile (Child Class)
Inheritance: Extends BP_BaseProjectile Unique Features: Fire-specific visual effects and materials
Collision Logic:
OnComponentHit event triggers on collision
Target Validation: Checks if hit actor matches filtering criteria
Self-Collision Prevention: Ignores caster to prevent self-damage
Damage Application: Uses AC_ResourceComponent for consistent damage handling
Cleanup: Destroys projectile and spawns impact effects
UI System Integration
WBP_MainHUD (Main Interface)
Purpose: Central UI controller managing all ability-related displays. Should be able to add more slots if needed, just need to add input and logic for more abilities. No art was made for this, so Icons were used are in the Engine already.
Event Binding System:
Connects to AC_Abilities event dispatchers on construction
OnCooldownStarted → OnCooldownUpdated: Cooldown initiation
OnAbilityActivated → OnAbilityUsed: Successful ability use feedback
OnAbilityFailed → OnAbilityFailedEvent: Error state handling
Dynamic Slot Management:
Uses PlayerAbilitySlots array to determine which abilities are equipped
Finds correct UI slot by matching ability IDs to array indices
Supports flexible ability arrangement without hardcoded mappings
WBP_AbilitySlot (Individual Slot Widget)

# Components:
Ability Icon: Visual representation of equipped skill
Cooldown Overlay: Semi-transparent cover during cooldown
Cooldown Text: Numeric countdown display
Keybind Indicator: Shows associated input key
Cooldown Animation System:
StartCooldown(): Initiates visual countdown with timer
OnCooldownTick(): Updates display every 0.1 seconds
StopCooldown(): Resets visual state and clears timers


Developed with Unreal Engine 5
