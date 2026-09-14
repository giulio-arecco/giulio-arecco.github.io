---
title: "Dragòn - OpenGL 2D Arcade Game"
date: 2024-03-05
summary: "A 2D scrolling arcade game built in C++ and OpenGL, featuring collectible power-ups, health and mana management, and unlockable levels with escalating difficulty."
tags: ["C++", "Game Development", "Computer Graphics"]
---

<div class="flex flex-wrap gap-2 mb-6">
  {{< button href="https://github.com/LienoPC/Dragon--2D-Scroller-Game" target="_blank" >}}
    {{< icon "github" >}} View Source Code
  {{< /button >}}

  {{< button href="https://www.youtube.com/watch?v=oxvbj1901CQ" target="_blank" >}}
    {{< icon "github" >}} Gameplay Demo
  {{< /button >}}
</div>


*Dragòn* is a 2D scrolling arcade game developed in C++ using OpenGL, GLFW, and GLM. Developed as a collaborative team project, the game combines 2D sprite rendering, keyboard navigation, projectile dodging, and stage progression.

**Project Overview:**

* **Role:** Game Developer / 3D Artist and Animator (Team of 3).
* **Context:** Computer Graphics / Game Development.
* **Responsibilities:** Menu and UI management, multi-part player hitboxes, fixed-rate animation control, player movement constraints, save state serialization, and 3D modeling/rigging in Blender.

## Contribution Overview

* **User Interface:** menu screens and UI buttons supporting hover states, cursor hit-testing, stage unlocking based on save data, and mouse-click handling.
* **3D Asset Creation and Fixed-Rate Animation:** creation of the 3D dragon model and wing-flap cycle in Blender, exported as a 2D sprite sequence and updated at a fixed 24 FPS.
* **Multi-Part Hitbox Geometry:** division of the dragon's shape into 5 distinct bounding boxes moving synchronously with the player.
* **Movement and Viewport Clamping:** directional WASD movement with sprint and precision speed modes, clamped to the window boundaries accounting for sprite padding.
* **Pause Logic and Save Serialization:** game pause state that freezes gameplay and compensates timers, alongside stage progression persistence to disk.

## Engineering Highlights

### Asset Modeling and Fixed-Rate Animation

The player character was modeled, rigged, and animated in Blender, then exported as an 8-frame sequential 2D sprite set. To avoid tying the animation speed directly to the rendering framerate, which would cause the wings to flap too quickly or slowly depending on frame rate, an accumulator-based animation controller was implemented. By accumulating delta time until reaching a fixed interval, the controller advances the sprite sequence in a bidirectional ping-pong loop locked at 24 FPS.

### Multi-Part Hitbox Geometry

Using a single bounding box for the dragon leaves large areas of empty space around the neck, wings, and tail, causing projectiles to register hits on transparent pixels. To match the character's silhouette, the dragon's shape was subdivided into 5 separate bounding rectangles, scaled relative to the sprite size. When the player moves, the translation vector is applied directly to the player coordinates and simultaneously propagated to all 20 vertices of the sub-boxes, keeping the hitbox positions aligned without allocating memory or recalculating offsets from scratch. For basic circular projectiles, a closest-point check computes squared distances against each box, avoiding square root operations.

### UI and Input Handling

The interface was built around dedicated `Menu` and `Button` classes to keep menu logic separate from game loops. Buttons manage idle and hover states, scaling slightly when hovered. Cursor collision is evaluated using an axis-aligned point-in-box check. To prevent misclicks, such as pressing down on one button and dragging the cursor off before releasing, the input handler records the active button on mouse-down and only fires the associated action if the cursor is still over that same button on mouse-up. The menu also queries completed stages to dynamically show medals and unlock subsequent levels.

### Movement Controls, Pause State, and Save System

Player movement integrates directional WASD inputs with a speed modifier that toggles between a standard speed, a sprint mode, and a precision slowdown mode for navigating tight bullet spaces. To keep the dragon within the window, the movement function checks target positions in advance, using visual offsets that account for transparent padding along the wing edges. A pause state machine halts actor updates and darkens the background scene using a shader uniform. While paused, the game timer accumulates the elapsed pause delta to prevent the in-game clock from advancing during pauses. Finally, level progression is written to disk as a text file (`save.txt`), recording the highest completed tiers across both campaign themes.
