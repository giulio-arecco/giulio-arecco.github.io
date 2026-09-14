---
title: "The Plague in Milan"
date: 2025-02-24
summary: "An interactive first-person experience bringing to life the dramatic events of 17th-century Milan as depicted in Alessandro Manzoni's \"The Betrothed\"."
tags: ["Unity", "C#", "Game Development"]
---

<div class="flex flex-wrap gap-2 mb-6">
  {{< button href="https://github.com/giulio-arecco/the-plague-in-milan" target="_blank" >}}
    {{< icon "github" >}} View Source Code
  {{< /button >}}

  {{< button href="https://giulio-arecco.itch.io/the-plague-in-milan" target="_blank" >}}
    {{< icon "itch-io" >}} Play on Itch.io
  {{< /button >}}
</div>

*The Plague in Milan* is a first-person 3D interactive narrative experience built in Unity, recreating the historical and social environment of 17th-century Milan during the plague epidemic described in Alessandro Manzoni's *The Betrothed*. It was developed as a collaborative project.

**Project Overview:**
*   **Role:** Game Programmer, System Designer (Team of 4).
*   **Context:** Virtual Reality Project.
*   **Responsibilities:** Character and camera physics synchronization, input normalization, narrative progression architecture, spatial NPC interaction systems.

## Contribution Overview

Direct engineering contributions to the project encompass the following functional areas:

*   **First-Person Character and Physics Setup:** implementation and editor configuration of a `Rigidbody`-driven first-person player controller.
*   **Player Camera Synchronization:** elimination of camera jitter and visual stutter by synchronizing the first-person camera position with the engine's physics fixed update cycle.
*   **Input Management:** configuration of Unity's Input System, including action map context switching and vector processing across keyboard/mouse and gamepads.
*   **Modular Progression System:** an event-driven architecture structured around modular objective steps (dialogues, spatial locations, physical interactions), supporting both individual progression checkpoints and composite steps that require multiple sub-objectives to be completed in arbitrary order.
*   **PC Interaction and Dialogue System:** A complete conversational architecture combining ScriptableObject data containers, synchronized audio voiceovers, dynamic typewriter text streaming, proximity-based interruption, programmatic camera framing, and billboard UI.

## Engineering Highlights

### First-Person Controller and Camera Synchronization
Player movement is built on a `Rigidbody` physics setup configured to interact reliably with level geometry and spatial triggers. In physics-driven first-person controllers, discrepancies between the fixed-rate physics update and the variable-rate frame rendering frequently produce visible jitter and camera stuttering. To resolve this, the camera tracking logic was tied directly to the physics simulation lifecycle. This ensures stable head movement while preserving consistent physical collision responses against the environment.

### Input Management
To handle locomotion consistently across both keyboards and gamepads, raw input vectors are processed before driving character mechanics. Orthogonal keyboard inputs inherently produce a diagonal magnitude greater than 1, which is mathematically normalized to prevent diagonal speed inflation. Conversely, analog stick inputs preserve their fractional magnitude to retain nuanced walk/jog speeds. Sprint states are gated by evaluating forward vector thresholds, preventing unnatural sideways or backward sprinting while tolerating thumbstick drift. Additionally, a centralized manager coordinates action map switching, disabling look or movement actions while in menus or during scripted sequences.

### Modular Progression System
To handle story progression the architecture is structured around modular, event-driven steps. Individual steps can be grouped together into composite quest phases, which allows designers to create progression checkpoints that require multiple sub-objectives (such as inspecting several clues or visiting different locations) to be fulfilled before advancing, regardless of the order in which the player completes them. Dedicated controllers encapsulate distinct interaction paradigms (listening to dialogue end events, tracking physical prop interactions, or monitoring spatial triggers), keeping gameplay mechanics decoupled from the global state. Spatial trigger volumes remain disabled until their specific step becomes active, unhooking themselves immediately upon completion to avoid redundant evaluations.

### NPC Interaction and Dialogue System
When talking to an NPC, the character rotates toward the player using a planar projection that zeroes out pitch deltas, tracking the player at a controlled angular rate without abnormal tilting. Simultaneously, scripted camera rotations interpolate target angles via `Mathf.LerpAngle` across an easing curve to select the shortest angular path during look-lock. Conversational content is structured via ScriptableObjects into specific behavioral archetypes: one-shot exchanges, ambient loops, event notifications, and progression barriers. A centralized manager streams dialogue lines through an internal FIFO queue with typewriter pacing while triggering synchronized voiceover clips directly from the speaker's audio source. To maintain physical plausibility, interactions enforce proximity constraints: if the player moves beyond an interaction threshold, active conversations are interrupted and queues are cleared. In-world nameplates and prompts rely on billboard calculations executed in `LateUpdate` with the roll axis clamped to zero, preventing horizon tilting during oblique camera angles.

## Gallery

{{< gallery >}}
  <img src="img/the-plague-in-milan/the_plague_in_milan_2.jpg" alt="The Plague In Milan - Example Screenshot" class="grid-w33" />
  <img src="img/the-plague-in-milan/the_plague_in_milan_4.jpg" alt="The Plague In Milan - Example Screenshot" class="grid-w33" />
  <img src="img/the-plague-in-milan/the_plague_in_milan_5.jpg" alt="The Plague In Milan - Example Screenshot" class="grid-w33" />
  <img src="img/the-plague-in-milan/the_plague_in_milan_6.jpg" alt="The Plague In Milan - Example Screenshot" class="grid-w33" />
  <img src="img/the-plague-in-milan/the_plague_in_milan_8.jpg" alt="The Plague In Milan - Example Screenshot" class="grid-w33" />
  <img src="img/the-plague-in-milan/the_plague_in_milan_9.jpg" alt="The Plague In Milan - Example Screenshot" class="grid-w33" />
{{< /gallery >}}
