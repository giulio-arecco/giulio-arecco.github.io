---
title: "Action RPG Systems"
date: 2026-11-03
summary: "A third-person Action RPG architecture built in Unreal Engine 5.6 featuring combat mechanics, attribute systems, and enemy AI."
tags: ["Unreal Engine 5", "C++", "Game Development"]
featured: true
---

<div class="flex flex-wrap gap-2 mb-6">
  {{< button href="https://github.com/giulio-arecco/ue5-action-rpg" target="_blank" >}}
    {{< icon "github" >}} View Source Code
  {{< /button >}}
</div>

This project features a third-person Action RPG architecture developed in Unreal Engine 5.6 with C++ and Blueprint visual scripting. It combines melee combat, directional hit reactions, state-driven character behavior, enemy AI, procedural foot placement, attribute-based UI, interactive pickups, and destructible props.

The project follows the curriculum presented in Stephen Ulibarri's [Unreal Engine 5 C++ The Ultimate Game Developer Course](https://www.udemy.com/course/unreal-engine-5-the-ultimate-game-developer-course/). Alongside completing the course material, I used the implementation to study the underlying topics in more depth and expand selected mechanics, including combat state handling, enemy AI behavior, dodging attacks, and stamina management.

## Tech Stack

* **Engine:** Unreal Engine 5.6
* **Languages:** C++ (and Blueprint)

## Architectural Overview

The main systems are divided between a shared character foundation, player input and locomotion, enemy behavior, attributes and HUD widgets, items and pickups, and destructible actors with Chaos force fields. `ABaseCharacter` contains logic shared by the player and enemies, while `APlayerCharacter` handles player input, equipment, locomotion, and action rules. Enemy classes manage sensing, patrol and combat states, and the attribute and HUD classes connect gameplay values to on-screen and world-space widgets.
Event-driven multicast delegates connect attribute changes to HUD and world-space widgets, keeping health, stamina, and progression currencies (gold and souls) independent from their visual representations. Interfaces such as `IHitInterface` and `IPickupInterface` provide common interaction contracts without coupling systems to concrete actor types.

## Engineering Highlights

### State-Driven Combat and Directional Hit Reactions

Player behavior is governed by two state machines. The character state tracks whether the weapon is armed or disarmed, while the action state controls whether the character is free to act, attacking, dodging, equipping, reacting to a hit, or dead. This separation prevents incompatible inputs from interrupting an animation or bypassing a gameplay rule.

Dodging is available only while the character is unoccupied and grounded, and requires sufficient stamina. Once the cost is consumed, the character enters the dodging state and ignores incoming damage until the dodge ends, providing invulnerability frames. Equipment changes are handled separately: animation notifies move the weapon between the hand and spine sockets at the appropriate point in the equip or unequip montage.

Damage follows Unreal's standard damage pipeline. Weapon traces apply damage to the target, which updates its attribute component and either triggers a hit reaction or starts the death sequence. On death, movement, collision, overlap events, and ticking are disabled before the death montage plays; the actor is then removed after a configurable lifespan.

For hit reactions, the impact point is projected onto the character's horizontal plane and compared with its forward direction. Cross and dot products provide the signed sine and cosine components of that angle, which `Atan2` converts into a stable value across the full circle. The result selects one of four front, back, left, or right montage sections. Using `Atan2` avoids division by zero and the out-of-range or near-axis instability that can affect tangent- or inverse-cosine-based calculations.

### Animation-Aware Weapon Tracing

Melee hit detection is enabled only during animation-defined collision windows. Box traces positioned along the weapon blade detect impacts, filter self-hits and friendly fire, and remember already-hit actors so a single swing cannot apply repeated damage. Valid impacts pass through Unreal's damage pipeline and the shared hit interface, allowing characters and destructible actors to react consistently.

### Enemy AI and Combat Engagement

Enemies move through explicit patrolling, chasing, combat, and death states. Pawn sensing filters potential targets by validity, actor tags, and attribute state before transitioning into the chasing state. Patrol selection uses a randomized modulo offset so an enemy cannot immediately choose the same waypoint, while path-completion callbacks and randomized waits keep navigation behavior decoupled from the main update loop.

During attacks, Motion Warping keeps enemy root motion aligned with the player while maintaining a configurable offset from the target. Distance checks use squared magnitudes to avoid unnecessary square-root operations in frequently evaluated range and combat queries.

### Event-Driven Attributes and UI

`UAttributeComponent` owns health, stamina, gold amount, and souls amount while including clamping and delayed stamina regeneration logic. Multicast delegates allow the viewport HUD to update reactively to attribute changes.

The same separation is used for world-space health bars: enemies expose their attribute changes through events, while the widget component controls presentation and visibility based on patrol, combat, damage, and death states.

### Destructible Props

Weapons interact with the environment by creating radial strain fields that apply physical forces that can break Geometry Collections while attacking. Breakable actors disable further break notifications after their first fracture, stop affecting navigation, schedule debris cleanup, and can spawn randomized treasure drops.
